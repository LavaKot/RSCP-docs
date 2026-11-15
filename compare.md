# RSCP vs Python vs Rust — where it sits

RSCP borrows the ergonomics of Python and the shape of Rust on purpose:
the result reads like Python with braces and compiles like a small Rust
program.

| | Python | Rust | RSCP |
|---|---|---|---|
| semicolons | none | required | optional (newline ends the statement) |
| blocks | indentation | braces | braces (required) |
| condition parens | no | no | no (a syntax error since 2026.5.2) |
| bindings | implicit | `let` / `let mut` | `let` / `var` |
| functions | `def f(x):` | `fn f(x) { }` | `fn f(x) { }` or `fn f(x) -> expr` |
| lambdas | `lambda x: x` | `\|x\| x` | `\|x\| x`, `x => x` |
| loops | `for i in range(n)` | `for i in 0..n` | `for i in 0..n` |
| infinite loop | `while True:` | `loop { }` | `loop { }` |
| slices | `a[1:3]`, `a[::-1]` | `&a[1..3]` | `a[1:3]`, `a[::-1]` |
| branching | `match` (3.10+) | `match` | `match` (expression, with guards and ranges) |
| classes | `class A:` | `struct` + `impl` | `class A(Parent) { fn ... }` |
| bitwise | `\| & ^ << >>` | `\| & ^ ! << >>` | `\| & ^ ~ << >>` (Rust precedence) |
| types | dynamic | static, inferred | dynamic |
| memory | GC | ownership | refcounted `Rc` values |
| speed | interpreter | native code | bytecode VM (≈ C-style loop ≈ 180 ns/iteration) |

## When to pick RSCP

* You want a **single portable binary** with no runtime setup — closer to a
  compiled tool than to a Python distribution.
* You like Python's lightness but prefer **braces and explicit `let`/`var`**.
* You are learning how languages work: the whole compiler + VM is ~9 000
  lines of safe, dependency-free Rust you can read in one evening.
* You need small scripts with classes, modules and error handling without
  installing anything beyond one executable.

## When to pick something else

* You need a huge library ecosystem (CPython) or zero-cost abstractions and
  static types (Rust) — RSCP is a young beta and its library is its builtins.
