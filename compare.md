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
| branching | `match` (3.10+) | `match` | `match` (expression, guards, ranges, enum destructuring with compile-time exhaustiveness) |
| classes | `class A:` | `struct` + `impl` | `class A(Parent) { fn ... }` |
| bitwise | `\| & ^ << >>` | `\| & ^ ! << >>` | `\| & ^ ~ << >>` (Rust precedence) |
| types | dynamic | static, inferred | dynamic (+ enum sum types with exhaustiveness) |
| memory | GC | ownership | refcounted `Rc` values |
| speed | interpreter | native code | bytecode VM (a pure counted loop runs ~18 ns/iteration) |

## When to pick RSCP

* You want a **single portable binary** with no runtime setup — closer to a
  compiled tool than to a Python distribution.
* You like Python's lightness but prefer **braces and explicit `let`/`var`**.
* You are learning how languages work: the whole compiler + VM is ~9 000
  lines of safe, dependency-free Rust you can read in one evening.
* You need small scripts with classes, modules and error handling without
  installing anything beyond one executable.

## Honest limits (26.0.12)

The Rust-inspired surface does not (yet) carry all of Rust's semantics:

* **No static types** — `enum` gives exhaustiveness and destructuring, not
  type checking; a match on a non-enum still needs a catch-all.
* **Same-file enums only** for patterns and exhaustiveness — `import`
  resolves at runtime, so imported enums are ordinary values.
* **Local mutual recursion** — a local `fn` can only call siblings that are
  declared before it; use top-level functions for mutual recursion.
* **Closures capture by reference** (no `move`), single-threaded VM, no FFI,
  no bigint (`i64` with checked overflow everywhere).
* **Modules** export a snapshot of their top-level bindings; there is no
  `pub`/`private`, and a module function reaching for a module-level var by
  name at call time reads the importer's globals — declare locals in
  functions and pass state explicitly.
* **No package manager** — `import "file.rscp"` by relative path, tests are
  the golden cases in the repo.

## When to pick something else

* You need a huge library ecosystem (CPython) or zero-cost abstractions and
  static types (Rust) — RSCP is a young beta and its library is its builtins.
