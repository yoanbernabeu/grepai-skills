---
name: grepai-trace-callers
description: Find function callers with GrepAI trace. Use this skill to discover what code calls a specific function.
---

# GrepAI Trace Callers

This skill covers using `grepai trace callers` to find all code locations that call a specific function or method.

## When to Use This Skill

- Finding all usages of a function before refactoring
- Understanding function dependencies
- Impact analysis before changes
- Code navigation and exploration

## What is Trace Callers?

`grepai trace callers` answers: **"Who calls this function?"**

```
func Login(user, pass) {...}
        ↑
        │
┌───────┴───────────────────┐
│   Who calls Login()?      │
├───────────────────────────┤
│ • HandleAuth (auth.go:42) │
│ • TestLogin (test.go:15)  │
│ • CLI (main.go:88)        │
└───────────────────────────┘
```

## Basic Usage

```bash
grepai trace callers "FunctionName"
```

### Example

```bash
grepai trace callers "Login"
```

Output:
```
🔍 Callers of "Login"

Found 3 callers:

1. HandleAuth
   File: handlers/auth.go:42
   Context: user.Login(ctx, credentials)

2. TestLoginSuccess
   File: handlers/auth_test.go:15
   Context: result := Login(testUser, testPass)

3. RunCLI
   File: cmd/main.go:88
   Context: err := auth.Login(username, password)
```

## JSON Output

For programmatic use:

```bash
grepai trace callers "Login" --json
```

Output:
```json
{
  "query": "Login",
  "mode": "callers",
  "count": 3,
  "results": [
    {
      "file": "handlers/auth.go",
      "line": 42,
      "caller": "HandleAuth",
      "context": "user.Login(ctx, credentials)"
    },
    {
      "file": "handlers/auth_test.go",
      "line": 15,
      "caller": "TestLoginSuccess",
      "context": "result := Login(testUser, testPass)"
    },
    {
      "file": "cmd/main.go",
      "line": 88,
      "caller": "RunCLI",
      "context": "err := auth.Login(username, password)"
    }
  ]
}
```

## Compact JSON (AI Optimized)

```bash
grepai trace callers "Login" --json --compact
```

Output:
```json
{
  "q": "Login",
  "m": "callers",
  "c": 3,
  "r": [
    {"f": "handlers/auth.go", "l": 42, "fn": "HandleAuth"},
    {"f": "handlers/auth_test.go", "l": 15, "fn": "TestLoginSuccess"},
    {"f": "cmd/main.go", "l": 88, "fn": "RunCLI"}
  ]
}
```

## TOON Output (v0.26.0+)

TOON format offers ~50% fewer tokens than JSON:

```bash
grepai trace callers "Login" --toon
```

Output:
```
callers[3]:
  - call_site:
      context: "user.Login(ctx, credentials)"
      file: handlers/auth.go
      line: 42
    symbol:
      name: HandleAuth
      ...
```

> **Note:** `--json` and `--toon` are mutually exclusive.

## Extraction Mode

Symbols and calls are always extracted with regular expressions (the "fast" extractor used by `grepai watch` and `grepai index`); output reports `"mode": "fast"`. The `--mode` flag is deprecated since grepai 1.0.0: it is still accepted for existing scripts, prints a deprecation warning and changes nothing. Do not pass `--mode precise`.

## Configuration

Configure trace in `.grepai/config.yaml`:

```yaml
trace:
  enabled_languages:
    - .go
    - .js
    - .ts
    - .py
    - .php
    - .rs
```

To keep files (tests, generated code) out of the symbol index, use the top-level `ignore` list. `trace.mode` and `trace.exclude_patterns` never had any effect and were removed in grepai 1.0.0.

## Supported Languages

| Language | Extensions |
|----------|------------|
| Go | `.go` |
| JavaScript | `.js`, `.jsx` |
| TypeScript | `.ts`, `.tsx` |
| Vue SFC | `.vue` |
| Python | `.py` |
| PHP | `.php` |
| Lua | `.lua` |
| Java | `.java` |
| C/C++ | `.c`, `.h`, `.cpp`, `.hpp`, `.cc`, `.cxx`, `.hxx` |
| Rust | `.rs` |
| Zig | `.zig` |
| C# | `.cs` |
| F# | `.fs`, `.fsx`, `.fsi` |
| Pascal/Delphi | `.pas`, `.dpr` |
| Ruby | `.rb` |
| GDScript | `.gd` |

A config created by an older version may lack newer extensions (`.vue`, `.lua`, `.java`, F#, `.rb`, `.gd`): add them to `trace.enabled_languages` by hand.

## Use Cases

### Before Refactoring

```bash
# Find all usages before renaming
grepai trace callers "getUserById"

# Check impact of changing signature
grepai trace callers "processPayment"
```

### Understanding Codebase

```bash
# Who uses this core function?
grepai trace callers "validateToken"

# Find entry points to a module
grepai trace callers "initialize"
```

### Debugging

```bash
# Where is this function called from?
grepai trace callers "problematicFunction"
```

### Code Review

```bash
# Verify function usage before approving changes
grepai trace callers "deprecatedMethod"
```

## Handling Common Names

If your function name is common, results may include unrelated code:

### Problem

```bash
grepai trace callers "get"  # Too common, many false positives
```

### Solutions

1. Use more specific name:
```bash
grepai trace callers "getUserProfile"
```

2. Filter results by path:
```bash
grepai trace callers "get" --json | jq '.results[] | select(.file | contains("auth"))'
```

## Combining with Semantic Search

Use together for comprehensive understanding:

```bash
# Find what Login does (semantic)
grepai search "user login authentication"

# Find who uses Login (trace)
grepai trace callers "Login"
```

## Scripting Examples

### Bash

```bash
# Count callers
grepai trace callers "MyFunction" --json | jq '.count'

# Get caller function names
grepai trace callers "MyFunction" --json | jq -r '.results[].caller'

# Get file paths only
grepai trace callers "MyFunction" --json | jq -r '.results[].file' | sort -u
```

### Python

```python
import subprocess
import json

result = subprocess.run(
    ['grepai', 'trace', 'callers', 'Login', '--json'],
    capture_output=True,
    text=True
)

data = json.loads(result.stdout)
print(f"Found {data['count']} callers of Login:")
for r in data['results']:
    print(f"  - {r['caller']} in {r['file']}:{r['line']}")
```

### Same-Name Definitions

When several functions share the name you trace (methods of different classes, functions in different files), grepai picks the definition deterministically: first by file path, then by line. The CLI and the MCP tools make the same choice, and it no longer depends on the order in which files were indexed.

Pass the bare name: `Class::method` or other qualified names are not supported.

```bash
grepai trace callers "save"        # ✅
grepai trace callers "User::save"  # ❌ not supported
```

Each caller is resolved in the file (and at the declaration line) where its call was found, so two same-name callers in one file are reported separately.

## Common Issues

❌ **Problem:** No callers found
✅ **Solutions:**
- Check function name spelling (case-sensitive)
- Ensure file type is in `enabled_languages`
- Run `grepai watch` to update symbol index

❌ **Problem:** Too many false positives
✅ **Solutions:**
- Use more specific function name
- Add test or generated paths to the top-level `ignore` list
- Filter results with `jq`

❌ **Problem:** Missing some callers
✅ **Solutions:**
- Check if files are in ignore patterns
- Check the extension is in `trace.enabled_languages`
- Dynamic calls (callbacks, reflection) are not detected by the regex extractor

## Best Practices

1. **Use exact function name:** Case matters
2. **Check symbol index:** Run `grepai watch` first
3. **Use JSON for scripts:** Easier to parse
4. **Combine with search:** Semantic + trace = full picture
5. **Filter large results:** Use `jq` or grep

## Output Format

Trace callers result:

```
🔍 Callers of "Login"

Mode: fast
Language files scanned: 245

Found 3 callers:

1. HandleAuth
   File: handlers/auth.go:42
   Context: user.Login(ctx, credentials)

2. TestLoginSuccess
   File: handlers/auth_test.go:15
   Context: result := Login(testUser, testPass)

3. RunCLI
   File: cmd/main.go:88
   Context: err := auth.Login(username, password)

Tip: Use --json for machine-readable output
```
