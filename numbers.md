---
title: Primeval numbers
---

A method for writing any rational number without a base. Inspired by [quipu](https://en.wikipedia.org/wiki/Quipu) and the [alien language in the 2016 movie _Arrival_](https://blog.wolfram.com/2017/01/31/analyzing-and-translating-an-alien-language-arrival-logograms-and-the-wolfram-language/).

- There are four “knot” symbols, which represent multiplication by
  2&nbsp;(<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1.5 c 2 0 2 3 0 3 s -2 -3 0 -3" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>), 3&nbsp;(<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1 c 2 0 2 2 0 2 s -2 2 0 2 s 2 -2 0 -2 s -2 -2 0 -2" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>), 5&nbsp;(<svg xmlns="http://www.w3.org/2000/svg" height="24" width="24" viewBox="0 0 6 6" style="vertical-align: bottom"><path d="M 3 1 c 2 0 0 4 -1.5 4 s -0.5 -2.5 1.5 -2.5 s 3 2.5 1.5 2.5 s -3.5 -4 -1.5 -4" fill="none" stroke="currentcolor" stroke-width="0.25" /></svg>) and -1 (undecided).
- A broken ring represents the multiplication all the knots, swooshes and rings on it. One with no knots represents 1, those with a single knot represent 2, 3, 5 or -1, and those with multiple knots etc. represent composite numbers.
- Prime numbers larger than 5 are represented as 1 more than a composite number. The +1 here is indicated by closing the ring of the composite number.
- Repeated factors can be compressed using exponention, which is indicated by a ring that encloses another. The outer ring is the exponent.
- The "main ring" is indicated with a dot in the middle. The dot by itself represents 0.

Note:
- Only natural numbers are currently supported and the symbol for -1 has not been decided.

<label>Input number: <input id="number" /></label>
<pre id="factors"></pre>

<svg viewbox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <defs>
      <marker
      id="dot"
      viewBox="0 0 10 10"
      refX="5"
      refY="5"
      markerWidth="5"
      markerHeight="5">
      <circle cx="5" cy="5" r="5" />
    </marker>
  </defs>
  <path id="thread" stroke="currentcolor" stroke-width="0.25" fill="none" marker-end="url(#dot)"
 />
</svg>

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
        2: ['c', 2, 0, 2, 3, 0, 3, 's', -2, -3, 0, -3],
        3: ['c', 2, 0, 2, 2, 0, 2, 's', -2, 2, 0, 2, 's', 2, -2, 0, -2, 's', -2, -2, 0, -2],
        5: ['c', 2, 0, 0, 4, -1.5, 4,
            's', -0.5, -2.5, 1.5, -2.5,
            's', 3, 2.5, 1.5, 2.5,
            's', -3.5, -4, -1.5, -4],
    };

    function turn(path, angle) {
        const c = Math.cos(angle);
        const s = Math.sin(angle);
        const turn_pt = (x, y) => [x * c - y * s, x * s + y * c];
        
        const turned = [];
        for (let i = 0; i < path.length; ) {
            if (path[i] === 'a') {
                console.log('turning', path.slice(i));
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
        const offset = level !== 0 ? 1 : close ? 0.5 : 0.25;
        const angle = 2 * Math.PI / (factors.length + (level > 0 ? 1 : 0)) * (level % 2 ? -1 : 1);
        const radius = Math.max(inner, 3 * Math.sqrt(factors.length));

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
            return factor_path(prime, exp, 0);
        }

        const result = factors.flatMap(([prime, exp], i) => {
            const path = [];

            console.log('initial', offset, 0, dxy(offset, 0));
            if (i === 0 && offset) path.push(...arc, ...dxy(offset, 0));

            let main = factor_path(prime, exp, 1);
            if (i + offset) main = turn(main, angle * (i + offset));
            path.push(...main);

            if (i < factors.length - 1) {
                console.log('final 1', i + offset + 1, i + offset, dxy(i + offset + 1, i + offset));
                path.push(...arc, ...dxy(i + offset + 1, i + offset));
            } else if (close) {
                console.log('final 2', 0, i + offset, dxy(0, i + offset));
                path.push(...arc, ...dxy(0, i + offset));
            } else {
                console.log('final 3', factors.length - 1 + 2 * offset, i + offset, dxy(factors.length - 1 + 2 * offset, i + offset));
                path.push(...arc, ...dxy(factors.length - 1 + 2 * offset, i + offset));
                path.push('m', ...dxy(0, factors.length - 1 + 2 * offset));
            }

            return path;
        });


        console.log('Result', result);
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


</script>