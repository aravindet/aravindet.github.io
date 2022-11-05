---
title: Smush 2
---

# Basics

- Smush is a superset of JSON. Strings, numbers, booleans, null, objects and arrays work in a familiar way.
- Immutable data structures + mutable variable bindings: Every data structure is immutable, and there is no way to, e.g. update one property of an object. However, local variables, including those in closures, can be re-bound.

# Functions

Consider:

```
{ "hello": "world" }
```

In JS, you might say this is an object with one property. In Smush, it is a function that returns `"world"` when invoked with an argument `"hello"`. (And `null` when invoked with any other argument.)

This is basically how objects work in Smush: they are functions that are called with a key and return the corresponding value. (Smush objects are immutable, so there is no set or delete operation.)

Functions in Smush accept one argument and return one value, but both those can be objects or arrays so this isn't as limiting as it sounds.

Here is a slightly more complex function.

```
fib = {
    0: 1,
    1: 1,
    n: fib(n - 1) + fib(n - 2)
}
```

`fib` is a function that accepts an argument `n` and returns the n<sup>th</sup> fibonacci number. Note that `n` here is an identifier; to create a case where the argument is a literal `"n"`, it needs to be quoted.

Complex expressions can be broken down into parts separated by semicolons:

```
memo = {};
fib = {
    0: 1,
    1: 1,
    n (memo(n)):
    n: memo(n) -> {
        null:
            f = fib(n - 1) + fib(n - 2);
            memo = { n: f, ...memo };
            memo(n),
        f: f
    }
};
```

Formally, function literals consist of:
- The open brace `{`
- One or more _cases_, each of which consist of:
    - A _match/bind expression_
    - A colon `:`
    - A _value expression_
    - A comma `,`, which is optional for the last case in a function.
- The close brace `}`

Every Smush source file contains a single function. Every JSON file is a valid Smush source file.

# Value expression

These are expressions that return a value, and consist of a series of function calls that are evaluated left-to-right.

# Match/bind expressions



# Semantics

## Iterable
{ iter: iterator }
- `iter` is a symbol imported from builtin 'iter'

## Iterator
{ null: { value, done } }
- each call returns a different value as long as done is false

## Promise
{ fn: null }
- the callback is invoked once when the promise resolves

## Stream
{ fn: null }
- the callback is invoked once, when the next value becomes available
- the callback receives { value, done }