---
title: Smush
---


> This is just an idea for a programming language. There is no implementation yet.

Smush is a tiny programming language intended for embedding in other programs. This places it in the same general space as Lua. The design goals are:

- Tiny (in compiled size) interpreter
- Easy interopability with the host program
- Familiarity for programmers from a wide range of backgrounds
- Support long-running programs without a complex GC
- Support asyncrhonous IO with cooperative multi-tasking

A secondary use case is as a command interpreter (shell), with advanced stream processing capabilities.

It is tiny because it is designed to have a very small interpreter as a result of having very few concepts.

- There are functions. They accept and return exactly one value.
- There are (Lua-like) tables. They are special functions.
- There are strings and numbers. They are special tables.
- There are symbols and null. They are ordinary tables.

# Introduction

Consider the expression console.log("Hello"). This is valid Smush.

You might think console is a table, log is a function and "Hello"  is a string. You'd be right, but the lines between those things are a bit blurry. The same statement could also be written as:

- `console["log"]("Hello")`
- `console "log" "Hello"`
- `console.log.Hello`

What's happening here? console is a function that is called with the string "log". This returns another function which is called with the string "Hello".

# Tour

**Strings**. There are a few ways to write string literals:

- 'Hello' and "Hello" work as expected
- .Hello works for simple wordlike strings (defined later)
- 'Hello', "Hello", "'Hello'" etc. because why not.

**Parentheses.** Incidentally, (parens), [brackets] and {braces} are all interchangeable and have no semantic difference:

- ("Hello") and ['Hello'] are the same as "Hello"
- [ 1, 2, foo: 3 ] is the same as ( 1, 2, foo: 3 )
- However, (3,) is NOT the same as 3.

More about this later in the section on blocks.

**Identifiers.** Smush identifier names can be either wordlike ([a-z_][a-z0-9_]*) or symbolike  (composed entirely of punctuation) but cannot mix words and punctuation. All wordlike strings are valid identifiers - there are no reserved keywords. However symbolike identifiers are more restricted.

The following characters cannot be used at all:

- block delimiters ()[]{}
- string delimiters '"

In addition, the following tokens cannot be used, although their characters can be used in other identifiers (like how for is not a valid identifier in many languages, but fork is):

- comment prefix #
- wordlike string prefix .
- statement terminator ;
- field terminator ,
- binding operators = and :
- function operator ->
- return operator <-
- spread / rest operator ...

**Expressions.** As functions in Smush accept exactly one argument, you call them by writing a parameter after them. foo val  calls foo with the argument val. foo bar baz calls foo with bar, then calls the result with baz.

All Smush expressions follow this pattern. Consider 1 + 2: + is a global variable containing a symbol ( an empty table that does nothing except being different from all other tables). The number 1  is called with this symbol as argument. That returns the addition function, which is then called with the number 2.

Note that Smush functions are all left associative and have no concept of operator precedence. (Which makes sense, it has no concept of operators either.) 1 + 2 * 3 in Smoosh will return 9, if you want it to work "normally" you should use 1 + (2 * 3). It's clearer anyway.

**Binding Expressions** (a = 3 or b: 4) creates a "variable" in the current scope and assign it a value. The = and : are interchangeable.

**Immutable.**  The binding operator is not an assignment operator; Smush values are immutable. Identifiers in an outer scope can be read from closures, but cannot be updated. Table properties can be read but not updated.

**Destructuring Expression.** The left-hand-side of a binding expression can be a destructuring expression. (x: x, y: y) = (x: 3, y: 4) and (x, y) = (3, 4) both result in x = 3 and y = 4.

You can have your cake and destructure it too: (foo: (bar: b) f) binds the subtable containing bar to f , while also binding the value of bar to b.

**Spread / Rest.** The ... expression syntax performs spread (in normal blocks) and rest (in destructuring blocks).

**Property shorthand.** When a local binding has the same name as a property in a table, you can use a shorthand: (:x, :y) for table construction and (:x, :y) = (x: 3, y: 4) for destructuring. Either binding operator (= or :) may be used.

**Function Expression.** For example, x -> x * x specifies a function. The left-hand-side can be a destructuring or matching expression.

**Conditionals and loops** are provided by standard library functions that are implemented in the interpreter. Note that lazily evaluated expressions such as loop bodies and conditional branches must be function expressions, i.e.

if condition thenFn elseFn

**Promises and streams** are also implemented in the standard library, using a thunk/callback pattern.

readfile 'example.log' (err, value) -> { ... do things }

**Blocks** The symbols ()[]{} define blocks. Blocks may contain:

- Statements. Expressions terminated by a ;
- Return statement with an optional value, e.g. <- 3;
- Field declarations. Expressions terminated by a ,
- Final Expression. An unterminated expression at the end

Blocks are expressions, and return a value when evaluated.

**Single expression blocks** return the value of that expression. For example, (3) returns 3.

**Explicit return values** work as expected. { a: 1 + 2; <- a } returns 3.

Table builder block. In all other cases, the block returns a table with all the evaluated field declarations. Any unterminated final expression is treated as a field.

- {} returns an empty table.
- [ 1, 2 ] creates an arraylike table.
- { x: 4, b: 6 } creates a maplike table.
- [ a: 1; x: 2, y = x + a ] returns [ x: 2, y: 3 ]
- [ 3, 5; <-; 4 ] returns [ 3, ]

Note that the use of {} for maplike tables and [] for listlike tables is merely convention.

**Modules.** An entire source file is considered a block. Source files can contain a single expression to return it, or can define multiple fields.

```
# Example program

# This program prints four random digits between 1 and 9 (inclusive,
# repetition allowed) and prompts the user for an input.
# The user should enter an RPN expression with these four (and no other)
# digits as well as the operators +, -, * and /, which evaluates to 24.

{ :log, :readLine } = require('console');
{ :parse } = require('number');
{ :sizeof } = require('table');
{ :break, :reduce, :filter } = require('iter');
{ :rand, :ceil } = require('math');

digits = [1..4].map () -> ceil(9 * rand());
log('The digits are' + digits.join(' '));
expr = readLine();

(stack, digits) = reduce(expr.split(), ([], digits),
  (_, c, (stack, digits)) -> {
    abort = msg -> {
      log(msg);
      break; # A symbol that tells iteration functions to stop
    }
    operate = fn -> {
      (...stack, a, b) = stack;
      !a | !b ? <- abort('Invalid expression.');
      stack + fn(a, b)
    };
    stack =
      c == '+' ? operate (a, b) -> a + b :
      c == '-' ? operate (a, b) -> a - b :
      c == '*' ? operate (a, b) -> a * b :
      c == '/' ? operate (a, b) -> a / b :
      {
        d = parseInt(c);
        left = filter(digits, (_, digit) -> digit == d);
        (sizeof left) == (sizeof digits) ? <- abort('Invalid digit ' + c);
        digits = left;
        stack + d
      };
    (stack, digits)
});

log(
  sizeof digits != 0 ? ('You did not use ' + digits.join(' ')) :
  stack.length != 1 ? 'Invalid expression' :
  stack[0] != 24 ? ('Result is ' + stack[0] + ', not 24.') :
  'Correct!'
);
```

# Grammar

This is just an early draft using the <a href="https://www.google.com/url?q=https://pest-parser.github.io/?bin%3D11yczz%23editor&amp;sa=D&amp;ust=1590658544763000">PEST Parser Editor</a>

```
WHITESPACE = _{ PATTERN_WHITE_SPACE }
COMMENT = _{ "#" ~ (!NEWLINE ~ ANY)* ~ NEWLINE }
reserved_punct = _{ "(" | "[" | "{" | "\" | "'" | "}" | "]" | ")" }
symbol = { !reserved_punct ~ PATTERN_SYNTAX }
reserved_operator = {
  ("=" | ":" | "." | ";" | "," | "->" | "<-") ~ !symbol }
wordlike = @{ XID_START ~ XID_CONTINUE* }
symbolike = @{ symbol+ }
identifier = { wordlike | !reserved_operator ~ symbolike }
escape_seq = { ( "n" | "r" | "t" ) }
chars_dbl = { (!("\" | "\\") ~ ANY | "\\" ~ ( "\" | escape_seq ))* }
chars_sgl = { (!("'" | "\\") ~ ANY | "\\" ~ ( "'" | escape_seq ))* }
number = @{ ASCII_DIGIT+ ~ ("." ~ ASCII_DIGIT+)? }
quoted_string = ${ "\" ~ chars_dbl ~ "\" | "'" ~ chars_sgl ~ "'" }
dotted_string = ${ "." ~ wordlike }
string = { quoted_string | dotted_string }
block = { "(" ~ block_body ~ ")" | "[" ~ block_body ~ "]" | "{" ~ block_body ~ "}" }
block_body = { (statement | property)* ~ expression }
statement = { expression ~ ";" }
property = { expression ~ "," }
shorthand = { ":" ~ wordlike }
dest_block = {
  "(" ~ dest_block_body ~ ")" |
  "[" ~ dest_block_body ~ "]" |
  "{" ~ dest_block_body ~ "}" }
dest_block_body = { (dest_property ~ ",")* ~ dest_property }
dest_property = {
  identifier |
  expression ~ (":" | "=") ~ (dest_block | identifier | dest_block ~ identifier ) |
  shorthand }
valuable = { number | string | identifier | block }
bindable = { identifier | dest_block }
call_expression = { valuable+ }
binding_expression = { bindable ~ ("=" | ":") ~ expression }
expression = {  binding_expression | call_expression }
```

# Smush by example

> These were written to an earlier draft and need to be updated.


<table>
<tbody>
<tr>
<th width='50%'>JavaScript</th>
<th width='50%'>Smush</th>
</tr>

<tr>
<td>
function factorial(n) {
  if (n === 1) return 1;
  return factorial(n - 1)
}
factorial(42)
</td>
<td>
factorial =
  n==1 -> 1,
  n -> factorial(n - 1);
factorial(42)
</td>
</tr>
<tr>
<td>
readFile('cfg.json', data => {
  cfg = parse(data)
  if (cfg.debug) print(data);
})
</td>
<td>
readFile('cfg.json', data -> {
  cfg = parse(data)
  cfg 'debug' |>
    true -> print(data);
})
</td>
</tr>
<tr>
<td>
// Prints the first N fibonacci
// numbers.
function fibonacci(n) {
  let m = 1, l = 0;
  for (let i = 1; i &lt;= n; i++) {
    print(m)
    [m, l] = [m + l, m];
  }
}
</td>
<td>
fibonacci = n -> {
  m: 1; l: 0;
  (1 .. n).each () -> {
    print(m);
    (m, l) = (m + l, m);
  };
}
</td>
</tr>

<tr>
<td>
console.log("Hello")
</td>
<td>
print "Hello"
# Also
print('Hello')
print.Hello
</td>
</tr>
<tr>
<td>
function a(x) {
 if (x) {
    bar()
  } else {
    baz()
  }
}
</td>
<td>
a =
  (=true) -> bar() :
 (=false) -> baz()

</td>
</tr>
<tr>
<td>
a(true)
</td>
<td>
a true # or a(true) or true | a
</td>
</tr>
<tr><td></td><td>
init = () -> {
  button.on.click event -> {
    event.shift |>
      true -> { <-- }
  }
}
</td></tr>
</tbody>
</table>
