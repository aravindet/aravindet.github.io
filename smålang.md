---
title: Smålang
marp: true
---

<style>
  pre, code {
    font-weight: 300;
    font-family: 'Cascadia Code', monospace;
  }
</style>

# Smålang

A **small**, structurally typed, embeddable **lang**uage for JSON-to-JSON transformation.

> Named after Småland, the kids’ play area at Ikea.

---

# Key ideas

---

## JSON is Smål

Smålang is a superset of JSON. Strings, numbers, booleans, null, objects and arrays work in a familiar way.

```py
{ "hello": "world" }
```

This is a Smål program returns `"world"` when invoked with the argument `"hello"` and `void` when invoked with any other argument.

---

## Types are values

Smålang values may be _abstract_ or _concrete_. An abstract value is very similar to a “type” — it defines the set of concrete values that “match” it.

Unlike types, Smålang abstract values are first-class: they can be bound to names, passed to and returned from functions, etc.

```py
{ "x": int, "y": int }
# "int" is an abstract value that matches all concrete integers
```

---

## Objects and functions are tables

All values in Smålang can be called, like functions; what that does varies.

When JSON objects are called with string keys as arguments, they return the corresponding values. Smålang’s function syntax is a generalization of this: keys can be anything, including abstract values.

```rb
fibonacci <- {
    0: 1,
    1: 1,
    int -> n: fibonacci(n - 1) + fibonacci(n - 2)
    # The key here is "int"; we’ll discuss the "-> n" soon.
}
```

---

## Smålang is functional

**Every value is immutable.** There is no way to, e.g. update one property of an object; you just construct a new one.

**Every function is pure**: outputs depend only on inputs. `now` is fixed for the duration if the computation. There is no `random` or IO.

**First-class functions.** They can be bound to names, passed to and returned from functions, etc.

**Lexical closures.** Inner functions can use bindings from outer functions.

---

# Syntax

---

## Smålang has:

- Everything in JSON: `"`, `{`…`}`, `[`…`]`, `:`, `,`
- The `->` and `<-` operators for binding values to names
- Function calls: `sqrt 9` (or `9!`)
- Parentheses for specifying order-of-operations
- Semicolons to end declarations
- The type manipulation operators `&`, `|` and `?`

And that’s pretty much it!

---

## What makes it Smål?

It _doesn’t_ have:

- Any keywords or operators. `true`, `null`, `int`, `+`, `=`, `<=` etc. are all functions from the standard library.
- Conditionals (`if`, `switch`, `match`) and loops (`for`, `while`).
- Complex operator precedence rules.
- Exception handling (`try`…`catch`)
- Nominal typing, inheritance, classes, interfaces, traits, …

---

## Identifiers and operators

- Smålang has typical identifiers like `foo_Bar12`. All words are valid identifiers, there are no reserved keywords.
- It also allows defining operators, like `+` and `<=`. They may not contain the characters `{`, `}`, `:`, `"`, `,` and `;`, and must not _be_ `->`, `<-`, `&`, `|` or `?`.
- Mixing characters from the identifier and operator character sets isn’t allowed. A string like `foo-bar` is parsed as three identifiers `foo`, `-` and `bar`. 
- TODO: Specify exact characters in line with [Unicode TR31](https://www.unicode.org/reports/tr31)

---

## Anatomy a Smål function

Functions are made up of `,`-separated _cases_. Each case consists of:
- A match / bind expression terminated by `:`
- Zero or more _declarations_ terminated by `;`
- A value expression

---

## Match / **Bind**

We've already seen binding: it uses the `<-` and `->` operators and work symmetrically in both directions.

```rb
greeting <- "Hello"

"World" -> planet
```

---

## **Match** / Bind

In functions, we _match_ and bind at the same time:
```rb
{ int -> n : ... }, # Match any integer and bind it to the name "n"
```

You can “de-structure” and match on parts too.

```rb
{ { "age": int -> age, "name": string -> name, ...any } : ... }
```

---

## Match / Bind Sugar

```rb
{ a: ... }
```
- If `a` is already defined in the parent scope, match its value.
- Otherwise, matches any value and bind to `a`. (i.e. it desugars to `{ any -> a: ... }`), 

```rb
{ { a }: ... }
```
- If `a` is defined in the parent scope, this desugars to `{ { "a": a }: ... }`
- Otherwise, it desugars to `{ { "a": any -> a }: ... }`

---

## Rest / Spread

To specify to the “all remaining branches” of a function when constructing it or matching against it, use `|`. This is equivalent to the `...` operator in JavaScript.

```rb
{ x: { "foo": x } | some_object }
```
Here, “some object” is (shallow) merged with the object `{ "foo": x }`.

```rb
{ { foo: x } | a: ... }
```
Assuming `a` is not already in scope, this removes the property “foo” from the input and binds the remainder to `a`.

---

## Declarations

These help break down computations into separate steps, and bind intermediate values to names. An example:

```rb
# This function accepts an object with "Price" and "Qty" properties
# and returns the total with shipping.
{
    { "Price": num -> price, "Qty": num -> qty }:        # match & bind
        total <- price * qty;                            # declaration
        shipping <- total < 100 {: true: 10, false: 0 }; # declaration
        total + shipping                                 # return value
}
```

---

## Expressions

Expressions are sequences of values that are mostly evaluated left-to-right without arbitrary rules like PEDMAS.

```rb
1 + 1 * 2   # Unlike most other languages, this is 4.
1 + (1 * 2) # This is 3.
```

Given a sequence `foo bar baz`, Smålang will evaluate it as `(foo bar) baz`.

Parentheses can be used to alter the evaluation order.

---

## Operators

Normally, the function on the left is evaluated with the argument on the right; e.g. `factorial(3)` evaluates the function `factorial` with the argument `3`. However when there’s an operator, say `3!`, it does the reverse and evaluates the `!` function with the argument `3` instead.


In the absence of parentheses, runs of identifiers are evaluated before operators. E.g. `foo bar + baz qux` is evaluated as `((foo bar) +)(baz qux)`. The `+` operator is called with the result of invoking `foo` with the argument `bar`. The operator returns an anonymous function, which is then called with the result of calling `baz` with `qux`.

---

## Special values

- `void` represents the absence of a value (like JS `undefined`).
- numbers, booleans, `null` and `void` always return `void` when called
- strings concatenate when called: `"hello" 3` returns `"hello3"`
- arrays are basically objects with integer keys.

---

## &, | and ?

- Recap: abstract values are *sets* of concrete values
- When `foo` _matches_ `bar`, it means `foo` is a subset of `bar`
- `|` and `&` operators perform the union and intersection operations
- `?` with a boolean expression using bound values performs _refinement_, removing concrete values that make the expression false. 

  ```rb
  {
    int -> n ? n > 0 : ... # Do something with n, a positive integer
  }
  ```

---

# Tricks

---

## Infix operators

Here’s how the `+` operator is implemented in Smål. We use function currying
with a mix of postfix and prefix functions so we can write `1 + 1`.

```rb
+ <- {: num -> a : { num -> b :
    # Native code to add a and b
} }
```

---

## Regular expressions

Written as `re "foo.*"` or `ri"foo.*/gi"`. `re` is a function from the standard library. The string passed to it doesn’t have to be a literal.

Regexes can be used as match expressions:

```rb
{
  re"^a.*": "This string has "
}
```

---

## Iteration

The basic idea is to have functions like filter, map and reduce. Details TBD.

---

# Todo
- Refine the syntax and remove rough edges
- Build an interpreter
- Build a type analyzer to detect type errors
- Add an effect system for doing IO, exceptions, futures
