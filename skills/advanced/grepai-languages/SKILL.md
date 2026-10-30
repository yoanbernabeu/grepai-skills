---
name: grepai-languages
description: Supported programming languages in GrepAI. Use this skill to understand which languages can be indexed and traced.
---

# GrepAI Supported Languages

This skill covers the programming languages supported by GrepAI for indexing and call graph analysis.

## When to Use This Skill

- Checking if your language is supported
- Configuring language-specific settings
- Understanding trace capabilities per language
- Troubleshooting language-related issues

## Supported Languages Overview

GrepAI indexes files with a built-in list of extensions (source code, config and documentation). Other extensions can be added with `chunking.custom_extensions`. Binary files, minified files (such as `*.min.js`) and files larger than 1 MB are skipped.

### Full Support (Index + Trace)

| Language | Extensions | Index | Trace |
|----------|------------|-------|-------|
| Go | `.go` | ✅ | ✅ |
| JavaScript | `.js`, `.jsx` | ✅ | ✅ |
| TypeScript | `.ts`, `.tsx` | ✅ | ✅ |
| Vue SFC | `.vue` | ✅ | ✅ |
| Python | `.py` | ✅ | ✅ |
| PHP | `.php` | ✅ | ✅ |
| Ruby | `.rb` | ✅ | ✅ |
| Lua | `.lua` | ✅ | ✅ |
| Java | `.java` | ✅ | ✅ |
| C | `.c`, `.h` | ✅ | ✅ |
| C++ | `.cpp`, `.hpp`, `.cc`, `.cxx`, `.hxx` | ✅ | ✅ |
| Rust | `.rs` | ✅ | ✅ |
| Zig | `.zig` | ✅ | ✅ |
| C# | `.cs` | ✅ | ✅ |
| F# | `.fs` (trace also `.fsx`, `.fsi`) | ✅ | ✅ |
| Pascal/Delphi | `.pas`, `.dpr` | ✅ | ✅ |
| GDScript | `.gd` | ✅ | ✅ |

### Index Only (No Trace)

| Language | Extensions |
|----------|------------|
| Swift | `.swift` |
| Kotlin | `.kt` |
| Scala | `.scala` |
| Dart | `.dart` |
| Elixir | `.ex`, `.exs` |
| Erlang | `.erl` |
| Clojure | `.clj` |
| Haskell | `.hs` |
| OCaml | `.ml` |
| Elm | `.elm` |
| Nim | `.nim` |
| R | `.r` |
| Svelte | `.svelte` |
| Shell | `.sh`, `.bash`, `.zsh` |
| SQL | `.sql` |
| HTML / CSS | `.html`, `.css`, `.scss`, `.less` |
| Markdown / text | `.md`, `.txt` |
| Config | `.yaml`, `.yml`, `.json`, `.toml`, `.xml`, `.ini`, `.cfg`, `.conf`, `.env` |
| Infrastructure | `.tf`, `.hcl`, `.proto` |

Any other extension (`.kts`, `.mdx`, `.htm`, `.prisma`…) needs `chunking.custom_extensions`.

## Language Configuration

### Enabling/Disabling Languages for Trace

Only extensions listed in `trace.enabled_languages` get symbols extracted. The defaults:

```yaml
# .grepai/config.yaml
trace:
  enabled_languages:
    - .go
    - .js
    - .ts
    - .jsx
    - .tsx
    - .vue
    - .py
    - .php
    - .lua
    - .gd
    - .rb
    - .c
    - .h
    - .cpp
    - .hpp
    - .cc
    - .cxx
    - .hxx
    - .rs
    - .zig
    - .cs
    - .java
    - .fs
    - .fsx
    - .fsi
    - .pas
    - .dpr
```

A config created by an older version may lack newer entries (`.vue`, `.lua`, `.gd`, `.rb`, `.hxx`, F#): add them by hand. An empty list falls back to the defaults.

### Excluding Files

`trace.mode` and `trace.exclude_patterns` never had any effect and were removed in grepai 1.0.0. To keep files (tests, mocks, generated code) out of every index, including the symbol index, use the top-level `ignore` list or a `.grepaiignore` file (gitignore syntax):

```yaml
ignore:
  - "*_test.go"
  - "*.spec.ts"
  - "**/fixtures/**"
```

To keep test files searchable but ranked lower, use search boosting penalties instead (see the `grepai-search-boosting` skill).

## Language-Specific Tips

### Go

**Trace accuracy:** Excellent. Go's explicit syntax makes tracing very reliable.

### JavaScript/TypeScript

**Trace accuracy:** Good. Some dynamic patterns may be missed. `*.d.ts` files are type declarations; ignore them if they add noise.

### Vue (Single-File Components)

`.vue` files are compiled with Node.js before indexing and tracing, and results are mapped back to the original lines:

```yaml
framework_processing:
  enabled: true
  mode: auto        # auto | require | off
  node_path: node   # Node.js command name or absolute path outside the project
  frameworks:
    vue:
      enabled: true
```

- `node_path` accepts only a command name resolved through `PATH` (`node`, `nodejs`, `node20`, `node.exe`) or a local absolute path outside the project and its git repository. Any other value (relative path, path inside the project or its repository, missing path…) is ignored with a warning, and grepai uses `node` from `PATH`. A Node.js installed inside the project (e.g. `./node_modules/.bin/node`) is no longer used.
- Without Node.js or `@vue/compiler-sfc`, grepai logs a warning and falls back to the `<script>` blocks.
- **Trust:** the compilation loads `@vue/compiler-sfc` from the project's `node_modules`. Only run grepai on repositories you trust, or set `framework_processing.frameworks.vue.enabled: false`.

### Python

**Trace accuracy:** Good. Dynamic imports and decorators may be missed. Comments and triple-quoted strings are handled, and Python keywords are not reported as calls.

### Ruby

**Trace accuracy:** Good. Methods (`def`, `def self.`, names ending in `?`, `!` or `=`), classes and modules are extracted. Method calls (`receiver.method`) are found with or without parentheses, but a bare call without parentheses (`route 'GET', path`) is not, because a regex cannot tell it apart from a variable.

### C/C++

**Trace accuracy:** Good. Macros and templates may affect accuracy.

### Rust

**Trace accuracy:** Excellent. Rust's explicit syntax aids accurate tracing.

### PHP

**Trace accuracy:** Good. Magic methods may not be fully traced.

### Java

**Trace accuracy:** Good. Reflection-based calls may be missed.

### C#

**Trace accuracy:** Good. Delegates and events may be partially traced.

## Multi-Language Projects

Trace only the languages you need, and keep tests out of the indexes if you don't want them:

```yaml
trace:
  enabled_languages:
    # Backend (Go)
    - .go
    # Frontend (TypeScript)
    - .ts
    - .tsx

ignore:
  - "*_test.go"
  - "*.spec.ts"
```

SQL and other index-only files are still searchable; they just have no call graph.

## Index vs Trace Explained

### Index (Semantic Search)

- Works on the built-in extensions plus `chunking.custom_extensions`
- Code is chunked and embedded
- Enables semantic search
- No language-specific parsing required

### Trace (Call Graphs)

- Requires **language-specific parsing**
- Extracts function definitions and calls
- Builds caller/callee relationships
- Always uses regular expressions: the `--mode` flag of `grepai trace` is deprecated and has no effect (output always reports `"mode": "fast"`)

## Adding Custom Extensions

Index extensions outside the built-in list with `chunking.custom_extensions` (leading dot required). They are indexed for search, not traced:

```yaml
chunking:
  custom_extensions:
    - .tengo
    - .prisma
```

The watcher uses the same list, so these files stay up to date.

## File Type Detection

GrepAI uses file extensions for detection. It does NOT use:
- Shebangs (`#!/usr/bin/env python`)
- File content analysis
- .editorconfig

## Best Practices

1. **Enable only needed languages:** Faster trace building
2. **Ignore generated code:** Cleaner search and trace results
3. **Update old configs:** Add newer extensions to `trace.enabled_languages`
4. **Match your tech stack:** Configure based on your actual languages
5. **Untrusted Vue repositories:** Disable `framework_processing.frameworks.vue`

## Checking Language Support

```bash
# Check what's being indexed
grepai status
```

## Common Issues

❌ **Problem:** Files not being indexed
✅ **Solution:** Check the file isn't ignored (`.gitignore`, `.grepaiignore`, `ignore`) and that its extension is built in or in `chunking.custom_extensions`

❌ **Problem:** Trace missing for language
✅ **Solution:** Ensure the extension is in `trace.enabled_languages` (older configs lack newer languages)

❌ **Problem:** `.vue` files only partly traced
✅ **Solution:** Install Node.js and `@vue/compiler-sfc` in the project; check the `node_path` warning in the logs

❌ **Problem:** Wrong language detected
✅ **Solution:** GrepAI uses extensions only; rename files if needed

## Output Format

Language support summary:

```
📚 GrepAI Language Support

Full Support (Index + Trace):
- Go, JavaScript, TypeScript, Vue, Python, PHP, Ruby, Lua, Java
- C/C++, Rust, Zig, C#, F#, Pascal/Delphi, GDScript

Index Only (No Trace):
- Swift, Kotlin, Scala, Dart, Elixir, Haskell, ...
- Shell scripts, SQL, HTML, CSS
- Config files (YAML, JSON, TOML)
- Documentation (Markdown)

Your config enables trace for:
- .go, .js, .ts, .py
```
