# Built-in functions

The complete standard library — everything ships inside the `rscp` binary,
nothing to install.

## Output and conversion

| Function | Behavior |
|----------|----------|
| `print(a, …)` / `println(a, …)` | print values space-separated, end the line (`println` is an alias of `print`) |
| `write(a, …)` | the same, without the newline |
| `str(v)` | value → string (`1.0` stays visibly `1.0`) |
| `int(v)` | float truncates toward zero; string parses (`int("42")`) |
| `float(v)` | int or string → float |
| `type(v)` | type name string (`"int"`, `"string"`, `"array"`, `"map"`, `"function"`, or the class name) |
| `len(v)` | string length (chars), array or map size |

## Numbers

| Function | Behavior |
|----------|----------|
| `abs` `min` `max` `sqrt` `pow` | numeric helpers (`pow` returns float, like C++) |
| `rand()` | pseudo-random float in `[0, 1)` |
| `clock()` | seconds since Unix epoch as float |
| `ord(c)` / `chr(i)` | character ↔ Unicode code point |

## Arrays and maps

| Function | Behavior |
|----------|----------|
| `push(arr, v)` / `pop(arr)` | array stack operations |
| `keys(map)` | **sorted** array of keys (deterministic iteration) |
| `has(map, key)` | key-presence test |
| `contains(arr, v)` | member test |

## Strings

| Function | Behavior |
|----------|----------|
| `upper` `lower` `trim` | case and whitespace (`upper("héllo")` is `HÉLLO`) |
| `split(s, sep)` / `join(arr, sep)` | string ↔ array |
| `replace(s, from, to)` | replace all occurrences |
| `contains(s, sub)` | substring test |
| `starts_with` / `ends_with` | prefix and suffix tests |
| `slice(x, a, b)` | substring or subarray; negative bounds, `b` exclusive |
| `find(s, sub)` | first char index of `sub`, or `-1` |
| `repeat(s, n)` | `repeat("ab", 3)` is `ababab` |

A quick taste:

```rust
println(upper("héllo"))                  # HÉLLO
println(split("a,b,c", ","))             # ["a", "b", "c"]
println(join(["a", "b"], "-"))           # a-b
println(replace("banana", "na", "NA"))   # baNANA
println(find("banana", "na"))            # 2 (first index, or -1)
println(slice("hello world", 0, 5))      # hello
println(slice([1, 2, 3, 4], -2, 99))     # [3, 4]
```

## Control and failure

| Function | Behavior |
|----------|----------|
| `assert(cond, msg?)` | runtime error if `cond` is false (catchable) |
| `panic(msg?)` | abort with a message (catchable with `try`) |

## Operators

| Precedence (high → low) | Operators                                  |
|-------------------------|--------------------------------------------|
| 1                       | `()` call, `[]` index                      |
| 2                       | `**` (right-associative, like Python)      |
| 3                       | unary `! - ~`                              |
| 4                       | `* / %`                                    |
| 5                       | `+ -` (`+` also concatenates strings/arrays)|
| 6                       | `<< >>`                                    |
| 7                       | `&`                                        |
| 8                       | `^`                                        |
| 9                       | `\|`                                       |
| 10                      | `< <= > >=`                                |
| 11                      | `== !=`                                    |
| 12                      | `&&`                                       |
| 13                      | `\|\|`                                     |
| 14                      | `..`, `..=` (ranges, `for` iterables only)  |
| 15                      | `? :` (right-associative)                  |

Like Rust, the bitwise operators bind *tighter* than comparisons
(`a & mask == 0` means `(a & mask) == 0`) — the opposite of C's infamous
precedence trap. `&`, `|`, `^` work on two ints or two bools; `~` inverts
an int or a bool; shifts require the amount in `0..64`. Integer literals
come in four bases: `42`, `0x2A`, `0b101010`, `0o52` (with `_` separators).

Assignment (`=`, `+=`, `-=`, `*=`, `/=`, `%=`, `**=`, `&=`, `|=`, `^=`, `<<=`,
`>>=`, `++`, `--`) is a *statement*, not an expression — no `if (x = 0)`
accidents. Like Python, `**` binds tighter than unary minus (`-2 ** 2` is
`-4`) and stays an exact `int` while the exponent is a non-negative `int`
(overflow is an error); anything else promotes to `float`. `==` compares
numbers, strings, bools and `null` by value (`1 == 1.0` is `true`); arrays,
maps and functions compare by identity. `&&`/`||` short-circuit and require
bool operands; so do `if`/`while` conditions. The word forms `and`, `or`,
`not` are exact synonyms of `&&`, `||`, `!` at the same precedence — note
that `not` binds as tightly as `!`, so `not a == b` means `(not a) == b`,
like Rust and unlike Python. Integer overflow and division by zero are
runtime errors, not UB.
