# Errors, exit codes and limits

## Compile-time errors

Compile-time errors point at the exact character:

```
$ rscp bad.rscp
syntax error[bad.rscp:3:9]: expected ';', found 'let'
   3 | let x = 1
     |         ^
```

## Runtime errors

Runtime errors carry a stack trace (innermost frame first):

```
$ rscp err.rscp
runtime error[err.rscp:1]: division by zero
   1 | fn inner() { return 1 / 0; }
stack trace (innermost first):
  1. fn 'inner' at line 1
  2. fn 'outer' at line 2
  3. fn 'main' at line 3
```

Runtime errors are catchable with `try` / `catch` (see the
[tutorial](tutorial.html#errors--try--catch--throw)); stack overflow and
internal errors are deliberately fatal.

## Typed errors (26.0.10.0)

Every catchable runtime error is a **typed object**: the `catch` binding is a
map with `code` (a stable machine-readable string), `message` (the human
text) and `detail` (structured data or null). `print(e)` still shows just the
message, Python-style.

```rust
try { let a = [1]; println(a[9]) }
catch e {
    println(e.code)              # IDX_RANGE
    println(e.message)           # index 9 out of bounds for array of length 1
    println(e.detail.index, e.detail.length)   # 9 1
}
```

### Error codes

| Code | Raised by | detail |
|------|-----------|--------|
| `DIV_ZERO` | `/ 0`, `% 0` | — |
| `INT_OVERFLOW` | integer overflow in `+ - * / % **`, `abs` | — |
| `IDX_RANGE` | index out of bounds (arrays, strings) | `{index, length}` |
| `MISSING_KEY` | `m[k]` with an absent key | `{key}` |
| `UNDEFINED_VAR` | reading an undefined variable | — |
| `IMMUTABLE` | assigning to a `let` binding | — |
| `ARITY` | wrong argument count (calls, `init`, methods) | `{expected, got}` |
| `TYPE` | wrong operand/argument type (`1 + "a"`, `if 1 { }`, …) | — |
| `UNKNOWN_METHOD` | unknown method/field, `super` misuse | — |
| `NO_ARM` | `match` with no matching arm | — |
| `ASSERT` | `assert(false)` | — |
| `PANIC` | `panic(msg?)` | — |
| `CONVERT` | `int("abc")`, `float("abc")` | — |
| `ARG` | invalid builtin arguments (`len(5)`, …) | — |
| `MATH` | domain errors (`pow(0, -1)`, zero slice step) | — |
| `USER` | `throw` of a non-error value (`throw "boom"`) | — |
| `RUNTIME` | rare internal catchable errors | — |

### throw and error()

`throw v` builds a typed error: error objects (from `error()` or a rethrow)
travel unchanged; a thrown map may carry its own `code`/`message` (the rest
lands in `e.detail`); anything else becomes `code "USER"` with the value's
text as the message. `error(code, message, detail?)` constructs one
explicitly:

```rust
throw error("PARSE", "bad token", {"pos": 3})
```

### Selective catch

A `try` may carry several `catch` clauses; the first one whose guard accepts
the error runs, and when all of them decline, the error is **rethrown
unchanged** to the outer scope:

```rust
try { risky() }
catch e code "DIV_ZERO" { println("divide by zero:", e) }
catch e where e.code == "TYPE" { println("bad type") }
catch e { println("other:", e.code) }
```

`code` and `where` are contextual keywords — they are only special directly
after `catch [name]`.

## Exit codes

| Exit code | Meaning (sysexits convention)      |
|-----------|-------------------------------------|
| 0         | success                            |
| 64        | bad command-line usage             |
| 65        | lexical / syntax / compile error   |
| 66        | input file could not be read       |
| 70        | runtime error                      |
| 141       | stdout closed early (SIGPIPE)      |

## Implementation limits

| Limit | Value |
|-------|-------|
| Locals per function | 255 |
| Parameters / arguments | 255 |
| Upvalues per function | 256 |
| Constants per function | 65 536 |
| Bytecode per function | 64 KiB |
| Call depth | 1024 frames |
| Expression nesting | 300 levels |
