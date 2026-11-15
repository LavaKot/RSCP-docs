# Tutorial — the language tour

This tour walks through the whole language in one sitting: the light style,
variables and types, control flow, `match`, modules, classes, error
handling, functions and closures, and strings with slices and f-strings.
Every fragment is a complete program you can save as `tour.rscp` and run
with `rscp tour.rscp`.

## The light style

Semicolons are **optional**: a newline after a complete expression ends the
statement (Go-style automatic semicolon insertion). Trailing commas in
multiline literals are welcome. Comments run from `#` to the end of the
line, Python style — `//` and `/* */` are no longer comment syntax, and
the compiler prints a migration hint when it meets them.

```rust
var total = 0
for i in 0..10 {                  # exclusive range: 0..9
    if i % 2 == 0 { continue }
    total += i
}
for i in 1..=5 { total += i }     # inclusive range: 1..5

while total > 5 { total -= 5 }    # no parentheses around conditions
loop {                            # the Rust infinite loop
    if total == 0 { break }
    total--
}
```

Functions come in two weights:

```rust
fn fib(n) -> n < 2 ? n : fib(n - 1) + fib(n - 2)   # arrow body: implicit return

fn fibIter(n) {                                    # block body: explicit return
    var a = 0
    var b = 1
    for _step in 0..n {
        let t = a + b
        a = b
        b = t
    }
    return a
}
```

Short closures (`|params| body`) return their expression implicitly —
the Rust style:

```rust
let double = |x| x * 2
let add = (a, b) => a + b          # arrow closures
let one = x => x + 1
let constant = || 42
let withBlock = |x| {              # a block body needs an explicit return
    var t = x + 1
    return t * 10
}
fn applyTwice(f, x) -> f(f(x))
println(applyTwice(|n| n * 3, 7))  # 63
```

Continuation rules are the same as in Go: a line that ends with an operator
or a comma continues; a line that ends with a complete value ends the
statement. `{` stays on the same line as the condition it belongs to:

```rust
var a = [                     # trailing commas are fine
    10,
    20,
]
var s = 1 +                   # '+' at the end: the next line continues
    2 +
    4
var grade = "F"
if total >= 100 {             # '{' on the condition's line
    grade = "A"
}
else if total >= 10 {         # 'else' may sit on its own line
    grade = "B"
}
```

Most of the 2026.1.0.0 syntax still parses — semicolons and C-style `for`
loops keep working, and the two styles mix freely in one file. Since
2026.5.2, though, parentheses around an `if`/`while` condition are a syntax
error (the compiler points at them and suggests the plain form). The
C-style loop is the only form that still wants its parentheses (and its
internal `;`); prefer ranges:

```rust
for (var i = 0; i < 10; i++) { ... }   # legacy, still works
for i in 0..10 { ... }                 # the modern way
```

## Variables: `let` vs `var`

`let` binds immutably (like Rust `let` / C++ `const`), `var` mutably. A
`let` always needs an initializer; `var x` alone is fine and declares `x`
as `null` — assign to it later.

```rust
let answer = 42                # cannot be reassigned
var count = 0                  # can
var later                      # null for now, assign later
count += 1                     # += -= *= /= %=
count++                        # ++ -- (statement forms)
```

At the top level, a `let`/`var`/`fn` name may be declared only once; inside
a scope, names can shadow outer ones:

```rust
var x = 1
{
    let x = 2                  # shadows the outer x
    println(x)                 # 2
}
println(x)                     # 1
```

Assigning to a `let` is a compile-time error for locals, and a runtime error
for globals:

```
error[prog.rscp:3:1]: cannot assign to immutable variable 'x' (declared with 'let')
```

Note that `let` fixes the *binding*, not the contents: `let a = [1]; a[0] = 5;`
is legal (the array is still mutable through itself).

## Values and types

RSCP is dynamically typed with seven user-visible types:

| Type      | Literals                    | Notes                                   |
|-----------|-----------------------------|-----------------------------------------|
| `int`     | `42`, `0xFF`, `1_000`       | 64-bit, checked overflow                |
| `float`   | `3.14`, `5.`, `.5`, `1e-3`  | 64-bit IEEE 754                         |
| `bool`    | `true`, `false`             | the only accepted condition type        |
| `string`  | `"hi"`, `'hi'`, `"""multi"""` | UTF-8; `f`/`r`/`b`/`u` prefixes; see below |
| `array`   | `[1, 2, [3]]`               | reference type, any mix of values       |
| `map`     | `{"a": 1, "b": [2]}`        | string keys, reference type             |
| `function`| `fn(x) { return x; }`, `\|x\| x` | closures; see below               |

`null` is the absence of a value; `nil` and `none` are synonyms. `type(v)`
returns a value's type name. Ints and floats mix freely in arithmetic
(`1 + 0.5` is `1.5`); `int / int` is integer division, C style (`7 / 2` is
`3`, `-7 / 2` is `-3`).

## Control flow

Bodies are a `{ }` block or a single statement on the same line:
`if ok print(x)`, `for i in 0..5 total += i` (a `{` that opens a block stays
on the condition's line). Conditions are written plain: `if x > 5 {`;
parentheses around the whole condition are a syntax error (parens that are
part of a bigger expression, like `if (a + b) * c > 0 {`, still parse).

```rust
if x < 10 { println("low") }
else if x < 100 { println("mid") }
else { println("high") }

while n > 0 { n-- }

loop { break }                            # infinite loop

for i in 0..10 { ... }                    # exclusive range
for i in 0..=9 { ... }                    # inclusive range
for v in [10, 20, 30] { println(v) }      # iterate array elements
for k in keys(mapVar) { println(k, mapVar[k]) }  # map keys, sorted
for var i in 0..3 { i = i * 10 }          # `var` makes the loop variable assignable
```

A range's end binds tightly: `for i in 0..n - 1` iterates `0..(n-1)`.
Ranges exist only as the iterable of a `for` loop — `let r = 0..5;` is a
compile error.

## Pattern matching: `match`

`match` is an **expression**: the value of the whole thing is the value of
the first arm that matches. Patterns are literals, alternations, integer
ranges, guards (`if`), and bindings that name the matched value. `_` or
`else` is the catch-all and must come last.

```rust
fn describe(n) -> match n {
    0 => "zero",
    1 | 2 => "one or two",       # alternation
    k if k < 0 => "negative",    # guard over the binding k
    3..=9 => "a digit",          # inclusive range (3..9 is exclusive)
    else => "many"               # catch-all: also spelled _
}

var size = match len(items) { 0 => "empty", 1..=5 => "small", _ => "big" }
```

An arm body is an expression, or a `{ }` block whose **last expression
statement is the arm's value** — handy for multi-step arms:

```rust
var label = match user {
    "admin" => "root",
    name => {
        let display = upper(name)
        "user: " + display
    }
}
```

Rules kept from Rust: a binding or catch-all only makes sense in the last
arm (earlier arms need an `if` guard, or the compiler rejects the match as
unreachable), and if nothing matches at runtime the match raises a
catchable `match: no arm matched the value` error. `break`/`continue`/`return`
inside an arm body do what they'd do in the enclosing loop or function.

## Modules: `import` and dot access

`import "file.rscp"` compiles and runs the module once (relative to the
importing file), then binds its top-level bindings — functions, constants,
anything — as an immutable map. Access members with dot syntax, which is
sugar for `["key"]` on any map:

```rust
import "math.rscp"             # binds the name from the file stem
println(math.double(21))       # 42
import "math.rscp" as m        # ...or pick your own name
println(m.golden)

var cfg = {"host": "localhost", "port": 8080}
println(cfg.port)              # dot reads...
cfg.port = 9090                # ...and writes; cfg["port"] still works
```

Diamond imports run the module body once (a cache keyed by path). Cycles
are a compile error. Exports are a **snapshot** of the module's top-level
bindings: pure functions and constants travel freely, and state works
through closures (shared upvalues) or arrays/maps exported by reference —
but a module function reaching for a module-level `var` by name after the
import is done sees the importer's globals, so keep mutable state inside
closures (`fn make_counter() { … }`) like in `examples/modules.rscp` in
the main repository.

## Classes

`class Name(Parent) { ... }` defines a class; the body holds `fn` methods.
`Name(args)` constructs an instance (calling `init` with `self` bound),
`self.x` reads and writes fields, `obj.method()` dispatches up the
inheritance chain, `super.method()` calls the parent implementation, and
`obj instanceof Class` tests the chain:

```rust
class Animal {
    fn init(name) { self.name = name }
    fn speak() -> "..."
    fn describe() -> f"{self.name} says {self.speak()}"
}
class Dog(Animal) {
    fn speak() -> "Woof"
}
let d = Dog("Rex")
println(d.describe())                   # Rex says Woof
println(d instanceof Animal, type(d))   # true Dog
```

Methods never list `self` in their parameters — the compiler injects it as
parameter 0. Overriding `init` replaces the parent's entirely; call the
parent yourself with `super.init(...)`. Instances print as
`<Dog instance>` and `type(instance)` returns the class name. Under the
hood a class is an immutable map (`{"__name__": ..., "__isclass__": true,
"__parent__": ..., methods}`) and an instance is a map holding a
`__class__` reference — so `keys()`, `has()` and map iteration work on
them, and classes compose with everything else in the language.

## Errors: `try` / `catch` / `throw`

Runtime errors are values you can catch: division by zero, out-of-bounds
indexes, missing map keys, failed assertions, `panic()` — and your own
`throw`. The `catch` binds the error's message text (or stays unnamed):

```rust
fn parse_int(s) {
    try { return int(s) }
    catch e { return -1 }        # "int: cannot convert 's' ..."
}

try { risky(0) } catch e { println("failed:", e) }
try { throw "custom boom" } catch { println("caught!") }
```

Errors unwind cleanly through any number of function frames into the
nearest enclosing `try` (with upvalues closed on the way), the catch
binding is an ordinary local (closures can capture it), and `return`,
`break` and `continue` all leave `try` blocks correctly. A `try` whose
body completes normally costs one opcode — a handler push and pop.

Deliberately **not** catchable: stack overflow and internal errors —
catching those would risk an immediate second failure inside the handler.
Compile-time errors are of course still compile-time. An uncaught error
exits with code 70 and a stack trace.

## Functions and closures

```rust
fn add(a, b) -> a + b             # arrow body
fn makeCounter() {                # closures capture by reference
    var count = 0
    fn inc() { count++; return count }
    return inc
}

var tick = makeCounter()
tick(); tick()
println(tick())                   # 3 — count survived makeCounter's return
```

* Functions are first-class: `let f = add; f(1, 2)`.
* Anonymous functions are expressions: `fn(x) { return x * x; }`, or the
  short form `|x| x * x`.
* Arguments are evaluated left to right; arity mismatches are runtime errors.
* Names resolve at call time, so mutual recursion needs no forward declaration.
* Parameters are assignable; the function binding itself is immutable.

Closures can nest arbitrarily deep — an inner function can capture a variable
from any enclosing function (see `examples/closures.rscp` in the main
repository).

## Programs and `main()`

A file runs as a script: top-level statements execute in order. If a global
function `main` is defined (with no parameters), it is called after the top
level finishes — so `fn main() { … }` works as the familiar entry point
while quick scripts can just write statements.

```rust
println("top level runs first")
fn main() { println("then main()") }
```

## Strings

Strings are UTF-8 and index by character: `s[0]` is a one-character string.
Concatenate with `+`, compare lexicographically with `<` and friends.

```rust
let s = "rscp" + "!"
println(s[0], len(s), s == "rscp!")    # r 5 true
```

Strings come in Python-style literal forms: either quote style,
triple-quoted multiline bodies, and 1-2 prefix letters:

```rust
let a = 'single quotes'
let m = """line one
line two"""                         # a real newline inside
let win = r"C:\path\no\escape"      # raw: backslashes stay verbatim
let bs = b"hi"                      # bytes: [104, 105], an array of UTF-8 bytes
let u = u"plain"                    # u is a no-op (Python compatibility)
println("\x41\u{1F600}")            # A😀 — \xNN and \u{...} escapes
```

The prefixes are `f` (interpolation), `r` (raw), `b` (bytes — the value is
an array of UTF-8 byte values), `u` (no-op), and `f` combines with `r`:
`fr"raw {x} \n stays"`. Escapes: `\n \t \r \0 \a \b \f \v \e`, `\xNN`,
`\u{...}` (1-6 hex digits), `\\`, `\'`, `\"`.

### Slices

Slices live directly in the index brackets, Python style — any part
optional, negative bounds count from the end, a negative step walks
backwards — and they cut arrays and strings alike:

```rust
let a = [0, 1, 2, 3, 4, 5]
println(a[1:3], a[::2], a[::-1], a[-1])   # [1, 2] [0, 2, 4] [5, 4, 3, 2, 1, 0] 5
println("abcdef"[1:4], "abcdef"[-4:-1])   # bcd cde
```

Plain indexing understands negatives too: `a[-1]` is the last element and
`a[-1] = 9` writes it (out of range is still an error). The `slice(x, a, b)`
builtin stays for the two-argument form.

### f-strings

Python-style interpolation embeds any expression in `{…}`; `{{` and `}}`
produce literal braces, and the holes nest — strings and even other
f-strings inside:

```rust
let name = "RSCP"
let n = 3
println(f"{name}: {n} squared is {n ** 2}")   # RSCP: 3 squared is 9
println(f"sum={1 + 2 * 3} nested={upper(f"{n}")}")  # sum=7 nested=3
```

The hole is parsed as a real expression at compile time, so mistakes inside
it are syntax errors with exact positions, and the pieces are joined at
runtime without building temporary arrays.

---

*Next: the [built-in functions](builtins.html) reference, the full
[grammar](grammar.html), or [RSCP vs Python vs Rust](compare.html).*
