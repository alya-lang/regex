# regex

[![CI](https://github.com/alya-lang/regex/actions/workflows/ci.yml/badge.svg)](https://github.com/alya-lang/regex/actions/workflows/ci.yml)
[![License](https://img.shields.io/github/license/alya-lang/regex?color=blue&label=License)](LICENSE)
[![Alya](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fregex%2Fmain%2Falya.toml&query=%24.package.alya-version&label=Alya&color=orange&prefix=%3E%3D)](https://github.com/alya-lang/alya)
[![Package Version](https://img.shields.io/badge/dynamic/toml?url=https%3A%2F%2Fraw.githubusercontent.com%2Falya-lang%2Fregex%2Fmain%2Falya.toml&query=%24.package.version&label=Version&color=brightgreen)](alya.toml)

High-performance, pure Alya regular expression engine with linear-time matching, named capture groups, lookarounds, backreferences, Unicode property classes, and global multilingual case-folding (Latin Extended, Cyrillic, Greek, Turkish, etc.).

---

## 🌟 Features

- ⚡ **Pure Alya & Fast**: Zero external C dependencies; compiles directly with `alyac` to native machine code (>190,000 matches/sec).
- 🌍 **Global Multilingual & Unicode**:
  - **Universal Case-Folding (`"i"` flag)**: Full bidirectional case-folding for German (`ä, ö, ü, ß`), French (`é, è, ê, à, ç`), Spanish (`ñ, á, é, í, ó`), Cyrillic / Russian (`А-Я` ↔ `а-я`), Greek (`Α-Ω` ↔ `α-ω`), Polish/Czech (`ł, ś, ź, ż, ć, č`), Scandinavian (`å, æ, ø`), and Turkish (`ç, ğ, ı, i, ö, ş, ü` ↔ `Ç, Ğ, I, İ, Ö, Ş, Ü`).
  - **Non-Cased Scripts**: Native UTF-8 matching for Chinese, Japanese, Korean, Arabic, Hebrew, Devanagari, etc.
  - **Unicode Property Classes**: `\p{L}` / `\p{Letter}` (letters across all Unicode blocks), `\p{N}` / `\p{Number}` (numeric digits).
- 🎯 **Comprehensive Pattern Syntax**:
  - **Literals & Escapes**: `\.`, `\\`, `\+`, `\*`, `\?`, `\^`, `\$`, `\(`, `\)`, `\[`, `\]`, `\{`, `\}`, `\|`, `\n`, `\r`, `\t`, `\0`.
  - **Hex & Unicode Escapes**: `\xHH` (e.g. `\x41`), `\uHHHH` (e.g. `\u0041`), hex ranges in character classes `[\x30-\x39]`.
  - **Character Classes**: Shorthands (`\d`, `\D`, `\w`, `\W`, `\s`, `\S`), custom ranges (`[a-z]`, `[0-9A-Fa-f]`, `[^0-9]`), negated classes.
  - **Anchors & Word Boundaries**: Line start `^`, line end `$`, word boundaries `\b`, non-word boundaries `\B`.
  - **Quantifiers**: Greedy (`*`, `+`, `?`, `{n}`, `{n,}`, `{min,max}`) and lazy modes (`*?`, `+?`, `??`).
  - **Alternation**: Multi-branch choices (`cat|dog|fish`).
  - **Capturing Groups**: Indexed groups `(...)` and non-capturing groups `(?:...)`.
  - **Named Capture Groups**: `(?<name>...)` and Python-style `(?P<name>...)`, accessible via `re::get_named(m, "name")`.
  - **Lookaround Assertions**:
    - Positive Lookahead: `(?=...)`
    - Negative Lookahead: `(?!...)`
    - Positive Lookbehind: `(?<=...)`
    - Negative Lookbehind: `(?<!...)`
  - **Backreferences**: Numbered backreferences `\1` through `\9`, and named backreferences `\k<name>`.
  - **Replacement Placeholders**: Numbered placeholders (`$0`, `$1`..`$9`) and named placeholders (`${name}`, `$<name>`).
- 🚩 **Standard Flags**:
  - `i`: Case-insensitive matching (with global multilingual UTF-8 case-folding).
  - `m`: Multiline mode (`^` and `$` match line starts and line ends).
  - `s`: Dotall mode (`.` matches newline `\n`).
- 🛠️ **Full Toolkit**: Pre-compilation (`compile`), one-liners (`test`, `find_pattern`), replacements (`replace`, `replace_all`), token splitting (`split`), and metacharacter escaping (`escape`).

---

## 📁 Project Architecture

```
regex/
├── alya.toml               # Package manifest
├── README.md               # Package documentation
├── src/
│   ├── lib.alya            # Public API facade & convenience functions
│   ├── types.alya          # Struct definitions (Regex, Match, Instruction, AstNode)
│   ├── utils.alya          # Character classification, Turkish case folding, word boundaries
│   ├── parser.alya         # Recursive descent AST parser
│   ├── compiler.alya       # Bytecode compiler emitting optimized VM instructions
│   └── vm.alya             # Virtual Machine execution engine with backtracking & slots
├── examples/
│   └── demo.alya           # Real-world usage demonstrations (9 feature categories)
├── tests/
│   ├── test_basic.alya     # Standard test suite (53 assertions)
│   ├── test_advanced.alya  # Advanced features test suite (38 assertions)
│   └── test_multilang.alya # Global multilingual & Unicode suite (19 assertions)
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

    # 2. Pre-compiled Regex with Named Capturing Groups
    let date_reg = re::compile("(?<year>\\d{{4}})-(?<month>\\d{{2}})-(?<day>\\d{{2}})")
    let m = re::find(date_reg, "Release date: 2026-09-16")
    if m != null
        say "Full Match: " + re::match_text(m)       # 2026-09-16
        say "Year:       " + re::get_named(m, "year") # 2026
        say "Month:      " + re::get_named(m, "month")# 09
        say "Day:        " + re::get_named(m, "day")  # 16
    end

    # 3. Named replacement placeholders (${day}, $<month>)
    let reordered = re::replace(date_reg, "2026-09-16", "$<day>/$<month>/$<year>")
    say reordered # "16/09/2026"

    # 4. Lookahead and Lookbehind assertions
    let px_reg = re::compile("\\d+(?=px)")
    let px_match = re::find(px_reg, "font-size: 16px")
    if px_match != null
        say "Size: " + re::match_text(px_match) # "16" (px is not consumed)
    end

    # 5. Backreferences (matching matching HTML tags)
    let tag_reg = re::compile("<(?<tag>\\w+)>(.*?)</\\k<tag>>")
    let valid_tag = re::is_match(tag_reg, "<b>hello</b>") # 1

    # 6. Unicode properties & Turkish case-folding
    let tr_reg = re::compile("şeker", "i")
    let tr_match = re::is_match(tr_reg, "ŞEKER") # 1
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
| `replace(reg, text, rep)` | `reg: Regex, text: string, rep: string` | `string` | Replaces the first match using template string (`$0`, `$1`, `${name}`, `$<name>`). |
| `replace_all(reg, text, rep)` | `reg: Regex, text: string, rep: string` | `string` | Replaces all matches using template string. |
| `split(reg, text, limit)` | `reg: Regex, text: string, limit = -1` | `[string]` | Splits `text` around matches of `reg`. |
| `get_named(m, name)` | `m: Match, name: string` | `string` | Retrieves the text of a named capture group. |

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
| `match_named(m, name)` | `m: Match, name: string` | `string` | Returns captured text for named group `name`. |
| `match_groups(m)` | `m: Match` | `[string]` | Returns array of all captured groups. |
| `match_span(m, idx)` | `m: Match, idx = 0` | `[start, end]` | Returns `[start, end]` offset pair for group `idx`. |

---

## 🧪 Running Tests & Benchmarks

Run all test suites automatically using `alyac`:

```bash
alyac test
```

Run individual test suites:

```bash
# Basic tests (53 assertions)
alyac run tests/test_basic.alya

# Advanced features (38 assertions)
alyac run tests/test_advanced.alya
```

Run performance micro-benchmarks:

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
   alyac test
   ```

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.