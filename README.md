# regex

[![CI](https://github.com/alya-lang/regex/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/regex/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/regex?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fregex%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fregex%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

High-performance, pure Alya regular expression engine with linear-time matching, character classes, greedy & lazy quantifiers, and capturing groups.

---

## 🌟 Features

- ⚡ **Pure Alya & Fast**: Zero external C dependencies; compiles directly with `alyac` to native machine code (>190,000 matches/sec).
- 🎯 **Rich Pattern Syntax**:
  - Literals & escapes (`\.`, `\\`, `\+`, `\*`, `\?`, `\^`, `\$`, `\(`, `\)`, `\[`, `\]`, `\{`, `\}`, `\|`, `\n`, `\r`, `\t`, `\0`).
  - Shorthand character classes (`\d`, `\D`, `\w`, `\W`, `\s`, `\S`).
  - Custom character classes and ranges (`[a-z]`, `[0-9A-Fa-f]`, `[^0-9]`, nested shorthands).
  - Anchors and word boundaries (`^`, `$`, `\b`, `\B`).
  - Quantifiers (`*`, `+`, `?`, `{n}`, `{n,}`, `{min,max}`) with greedy and lazy modes (`*?`, `+?`, `??`).
  - Alternations (`cat|dog|fish`).
  - Capturing groups (`(...)`) and non-capturing groups (`(?:...)`).
- 🚩 **Standard Flags**:
  - `i`: Case-insensitive matching.
  - `m`: Multiline mode (`^` and `$` match line starts and line ends).
  - `s`: Dotall mode (`.` matches newline `\n`).
- 🛠️ **Full Toolkit**: Pre-compilation (`compile`), one-liners (`test`, `find_pattern`), group replacements (`$0`, `$1`, `$&`), splitting (`split`), and metacharacter escaping (`escape`).

---

## 📁 Project Architecture

```
regex/
├── alya.toml               # Package manifest
├── README.md               # Package documentation
├── src/
│   ├── lib.alya            # Public API facade & convenience functions
│   ├── types.alya          # Struct definitions (Regex, Match, Instruction, AstNode)
│   ├── utils.alya          # Character classification, word boundaries, escape helpers
│   ├── parser.alya         # Recursive descent AST parser
│   ├── compiler.alya       # Bytecode compiler emitting optimized VM instructions
│   └── vm.alya             # Virtual Machine execution engine with capture tracking
├── examples/
│   └── demo.alya           # Real-world usage demonstrations
├── tests/
│   └── test_basic.alya     # Comprehensive 53-assertion test suite
└── benches/
    └── bench_basic.alya    # Micro-benchmarking suite
```

---

## 📦 Installation

Add `regex` to the `[dependencies]` section in your `alya.toml`:

```toml
[dependencies]
regex = { git = "https://github.com/alya-lang/regex", branch = "main" }
```

Or install it directly using the Alya package CLI:

```bash
alyac add regex --git https://github.com/alya-lang/regex --branch main
alyac install
```

---

## 🚀 Quick Start

```alya
import "regex" as re

function main()
    # 1. Quick pattern test
    if re::test("^[a-zA-Z0-9_]+$", "username_123") == 1
        say "Valid username!"
    end

    # 2. Pre-compiled Regex with Capturing Groups
    let date_reg = re::compile("(\\d{{4}})-(\\d{{2}})-(\\d{{2}})")
    let m = re::find(date_reg, "Release date: 2026-09-16")
    if m != null
        say "Full Match: " + re::match_text(m)       # 2026-09-16
        say "Year:       " + re::match_group(m, 1)   # 2026
        say "Month:      " + re::match_group(m, 2)   # 09
        say "Day:        " + re::match_group(m, 3)   # 16
    end

    # 3. Replacing with group placeholders ($1, $2)
    let name_reg = re::compile("(\\w+)\\s+(\\w+)")
    let reordered = re::replace(name_reg, "Ada Lovelace", "$2, $1")
    say reordered # "Lovelace, Ada"

    # 4. Token Splitting
    let tokens = re::split_pattern("[,;\\s]+", "apple, banana; cherry  date")
    # tokens -> ["apple", "banana", "cherry", "date"]
end

main()
```

---

## 📖 API Reference

### Compilation & Core Operations

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `compile(pattern, flags)` | `pattern: string, flags = ""` | `Regex` | Compiles a regular expression into an optimized bytecode object. |
| `regex(pattern, flags)` | `pattern: string, flags = ""` | `Regex` | Alias for `compile()`. |
| `is_match(reg, text)` | `reg: Regex, text: string` | `int (1/0)` | Checks whether the pattern matches anywhere within `text`. |
| `find(reg, text)` | `reg: Regex, text: string` | `Match / null` | Locates the first match in `text`. |
| `find_all(reg, text)` | `reg: Regex, text: string` | `[Match]` | Returns all non-overlapping matches found in `text`. |
| `replace(reg, text, rep)` | `reg: Regex, text: string, rep: string` | `string` | Replaces the first match using template string (`$0`, `$1`..`$9`). |
| `replace_all(reg, text, rep)` | `reg: Regex, text: string, rep: string` | `string` | Replaces all matches using template string. |
| `split(reg, text, limit)` | `reg: Regex, text: string, limit = -1` | `[string]` | Splits `text` around matches of `reg`. |

### Pattern One-Liner Functions

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `test(pattern, text, flags)` | `pattern, text, flags = ""` | `int (1/0)` | Quickly tests if `pattern` matches `text`. |
| `find_pattern(pattern, text, flags)` | `pattern, text, flags = ""` | `Match / null` | Finds first match of `pattern` in `text`. |
| `find_all_pattern(pattern, text, flags)` | `pattern, text, flags = ""` | `[Match]` | Finds all matches of `pattern` in `text`. |
| `replace_pattern(pattern, text, rep, flags)` | `pattern, text, rep, flags = ""` | `string` | Replaces first match of `pattern`. |
| `replace_all_pattern(pattern, text, rep, flags)` | `pattern, text, rep, flags = ""` | `string` | Replaces all matches of `pattern`. |
| `split_pattern(pattern, text, limit, flags)` | `pattern, text, limit = -1, flags = ""` | `[string]` | Splits `text` by `pattern`. |
| `escape(str_val)` | `str_val: string` | `string` | Escapes metacharacters for literal matching. |

### Match Helpers

| Function | Arguments | Returns | Description |
|---|---|---|---|
| `match_text(m)` | `m: Match` | `string` | Returns the entire matched substring. |
| `match_start(m)` | `m: Match` | `int` | Returns the zero-based starting character offset. |
| `match_end(m)` | `m: Match` | `int` | Returns the zero-based ending character offset. |
| `match_len(m)` | `m: Match` | `int` | Returns match length (`end - start`). |
| `match_group(m, idx)` | `m: Match, idx = 0` | `string` | Returns captured group text (`0` = full match, `1..n` = groups). |
| `match_groups(m)` | `m: Match` | `[string]` | Returns array of all captured groups. |
| `match_span(m, idx)` | `m: Match, idx = 0` | `[start, end]` | Returns `[start, end]` offset pair for group `idx`. |

---

## 🧪 Running Tests & Benchmarks

Run the test suite using `alyac`:

```bash
alyac run tests/test_basic.alya
```

Run the performance micro-benchmarks:

```bash
alyac run benches/bench_basic.alya
```

Run the interactive showcase:

```bash
alyac run examples/demo.alya
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository and create your feature branch:
   ```bash
   git checkout -b feat/my-feature
   ```
2. Ensure code is formatted with the canonical formatter:
   ```bash
   alyac fmt .
   ```
3. Run the automated test suite before opening a pull request:
   ```bash
   alyac run tests/test_basic.alya
   ```

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.