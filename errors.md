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
