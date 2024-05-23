---
title: Primeval numbers
---

A method for writing numbers using their prime factors instead of digits. Visual design inspired by the [Inca record-keeping technology quipu](https://en.wikipedia.org/wiki/Quipu) and the [alien language in the 2016 movie _Arrival_](https://blog.wolfram.com/2017/01/31/analyzing-and-translating-an-alien-language-arrival-logograms-and-the-wolfram-language/).

# Playground

<label>Input number: <input id="number" type="number" min="0" step="1" value="60" /></label>
<pre id="factors" style="height: 2rem; text-align: center;"></pre>

<svg viewbox="0 0 100 100" xmlns="http://www.w3.org/2000/svg"
    style="display: block; margin: 0 auto; max-height: calc(50vh - mod(50vh, 1rem)); max-width: calc(100% - mod(100%, 1rem));">
  <defs>
      <marker
      id="dot"
      viewBox="0 0 10 10"
      refX="5"
      refY="5"
      markerWidth="5"
      markerHeight="5">
      <circle cx="5" cy="5" r="5" stroke="none" fill="currentcolor" />
    </marker>
  </defs>
  <path id="thread" stroke="currentcolor" stroke-width="0.2" fill="none" marker-end="url(#dot)"
 />
</svg>

# How it works

<style>
    .nowrap {
        white-space: nowrap;
    }
</style>

- There are three _knots_, representing multiplication by
  <span class="nowrap">2<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1.5 c 1.5 0 2 3 0 3 s -1.5 -3 0 -3" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>,</span> <span class="nowrap">3<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1 c 1.5 0 2 2 0 2 s -2 2 0 2 s 2 -2 0 -2 s -1.5 -2 0 -2" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>,</span> <span class="nowrap">5<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1 c 2 0 0 4 -1.5 4 s -0.5 -2.5 1.5 -2.5 s 3 2.5 1.5 2.5 s -3.5 -4 -1.5 -4" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>.</span>
- An incomplete ring represents the product of all the knots and rings connected to it. With no knots it represents 1; with a single knot it represent 2, 3 or 5. With multiple knots and connected rings, it represents a composite number.
- Prime numbers larger than 5 are represented as a composite number + 1, by closing the ring of the corresponding composite number. Complete rings are only valid when the product of all their connected knots and rings is one less than a prime number.
- Repeated factors can be compressed using exponention, which is indicated by a ring that encloses another. The outer ring is the exponent, which is encoded the same way as regular numbers.
- The starting point of the first ring is indicated with a small teardrop shape, which indicates the direction along the string to read this number. The teardrop by itself represents 0.

## Future
A fourth symbol for -1 is planned. In addition to negative numbers, this will unlock fractions (-1 as exponent), n<sup>th</sup> roots (fractional exponents), complex numbers (-1 raised to fractional exponents) and their products.

<script>
    const known_primes = [5];

    function is_prime(number) {
        for (const prime of known_primes) {
            if (number === prime) return true;
            if (number % prime === 0) return false;
        }
        known_primes.push(number);
        return true;
    }

    function prime_factors(number) {
        if (number === 0 || number === 1) return number;

        const max = Math.sqrt(number);
        const factors = [];
        
        function add(factor) {
            let exponent = 0;
            do {
                number /= factor;
                exponent++;
            } while (number % factor === 0);
            
            if (factor > 5) factor = prime_factors(factor - 1);

            factors.push([factor, prime_factors(exponent)]);
        }

        function check_add(factor) {
            if (is_prime(factor) && number % factor === 0) add(factor)
        }

        if (number % 2 === 0) add(2);
        if (number % 3 === 0) add(3);
        if (number % 5 === 0) add(5);

        for (let i = 6; i < max; i += 6) {
            check_add(i + 1);
            check_add(i + 5);
        }

        if (number > 1) add(number);

        return factors;
    }

    function format(factors, root) {
        if (typeof factors === 'number') return factors;
        const inner = factors.map(([prime, exp]) => {
            const sup = exp === 1 ? '': `<sup>${format(exp, true)}</sup>`;
            return `${format(prime)}${sup}`;
        }).join('•');
        return root ? inner : `(${inner} + 1)`;
    }

    const knots = {
        0: ['l', 0.01, 0],
        1: ['a', 3, 3, 0, 1, 1, 0, 6, 'm', 0, -6],
        2: ['c', 1.5, 0, 2, 3, 0, 3, 's', -1.5, -3, 0, -3],
        3: ['c', 1.5, 0, 2, 2, 0, 2, 's', -2, 2, 0, 2, 's', 2, -2, 0, -2, 's', -1.5, -2, 0, -2],
        5: ['c', 2, 0, 0, 4, -1.5, 4,
            's', -0.5, -2.5, 1.5, -2.5,
            's', 3, 2.5, 1.5, 2.5,
            's', -3.5, -4, -1.5, -4],
    };

    function turn(path, angle) {
        if (angle === 0) return path;
        const c = Math.cos(angle);
        const s = Math.sin(angle);
        const turn_pt = (x, y) => [x * c - y * s, x * s + y * c];
        
        const turned = [];
        for (let i = 0; i < path.length; ) {
            if (path[i] === 'a') {
                turned.push(...path.slice(i, i + 6));
                i += 6;
            } else if (typeof path[i] !== 'number') {
                turned.push(path[i]);
                i += 1;
                continue;
            }
            turned.push(...turn_pt(path[i], path[i+1]));
            i += 2;
        }
        return turned;
    }

    function number_path(factors, inner = 0, level = 0, close = false) {
        if (typeof factors === 'number') return knots[factors];
        const offset = (level > 0 ? 0.5 : 0) + (close ? 0.5 : 0.25);
        const num_points = factors.length + (level > 0 ? 1 : 0);
        const angle = 2 * Math.PI / num_points * (level % 2 ? -1 : 1);
        // Radius of the circumcircle of a regular polygon with num_points sides of length 6:
        const radius = Math.max(inner, 3 / Math.sin(Math.PI / Math.max(2, num_points)));

        const xy = (i) => [radius * Math.sin(angle * i), -radius * Math.cos(angle * i)];
        const dxy = (i, j) => [
            radius * (Math.sin(angle * i) - Math.sin(angle * j)),
            -radius * (Math.cos(angle * i) - Math.cos(angle * j))
        ];
        const arc = ['a', radius, radius, 0, 0, 1 - level % 2];

        const factor_path = (prime, exp, bump) =>
            turn(number_path(prime, inner, level + bump, true), Math.PI).concat(
                exp === 1 ? [] :
                turn(
                    number_path(
                        typeof exp === 'number' ? [[exp, 1]] : exp,
                        3, level + bump, false
                    ),
                    Math.PI
                )
            );

        if (factors.length === 1 && Array.isArray(factors[0][0])) {
            const [prime, exp] = factors[0];
            return turn(factor_path(prime, exp, 0), Math.PI);
        }

        const result = factors.flatMap(([prime, exp], i) => {
            const path = [];

            if (i === 0 && offset) path.push(...arc, ...dxy(offset, 0));

            let main = factor_path(prime, exp, 1);
            if (i + offset) main = turn(main, angle * (i + offset));
            path.push(...main);

            if (i < factors.length - 1) {
                path.push(...arc, ...dxy(i + offset + 1, i + offset));
            } else if (close) {
                path.push(...arc, ...dxy(0, i + offset));
            } else {
                path.push(...arc, ...dxy(factors.length - 0.75 + offset, i + offset));
                path.push('m', ...dxy(0, factors.length - 0.75 + offset));
            }

            return path;
        });

        return result;
    }

    const input_el = document.querySelector('input#number');
    const output_el = document.querySelector('#factors');
    const thread_el = document.querySelector('#thread');
    
    input_el.addEventListener('input', (event) => {
        const number = parseInt(event.target.value);
        const factors = prime_factors(number);
        output_el.innerHTML = format(factors, true);
        // + '\n' + JSON.stringify(factors, null, 2);
        thread_el.setAttribute('d', 'M 50 50 ' + number_path(factors).join(' '));
    });

    input_el.dispatchEvent(new InputEvent('input'));


</script>