# RSCP — documentation

**A scripting language as light as Python, shaped like Rust — implemented
from scratch in Rust.**

```rust
# FizzBuzz, the RSCP way — no ';' anywhere.
for i in 1..=30 {
    if i % 15 == 0      { println("FizzBuzz") }
    else if i % 3 == 0  { println("Fizz") }
    else if i % 5 == 0  { println("Buzz") }
    else                { println(i) }
}
```

This is the documentation project for RSCP: tutorials, the full language
tour, grammar and reference. The compiler, the VM and the releases live in
the main repository: **[LavaKot/RSCP](https://github.com/LavaKot/RSCP)**.

## Learn

* **[Tutorial — the language tour](tutorial.html)** — syntax, variables,
  types, control flow, `match`, modules, classes, error handling, closures,
  strings, slices, f-strings.
* **[Built-in functions](builtins.html)** — the complete standard library:
  `len`, `split`, `replace`, `keys`, `assert`, …
* **[Grammar](grammar.html)** — the full EBNF and the semicolon-insertion
  rules.
* **[Errors, exit codes and limits](errors.html)** — how failures are
  reported and what the implementation limits are.
* **[RSCP vs Python vs Rust](compare.html)** — where RSCP sits between the
  two.

## Install

Grab a build from the
[releases page](https://github.com/LavaKot/RSCP/releases): a Windows
installer (`*-setup.exe`), a self-extracting Linux installer
(`*-installer.sh`), a ready-to-run Windows zip, full sources and the VS Code
extension. Then:

```
rscp version
rscp examples\fizzbuzz.rscp      # Windows
rscp examples/fizzbuzz.rscp      # Linux / macOS
```

## Bugs

Found a bug? Please
[open an issue](https://github.com/LavaKot/RSCP/issues) with the smallest
program that reproduces it, what you expected, what happened, and your
`rscp version`.
