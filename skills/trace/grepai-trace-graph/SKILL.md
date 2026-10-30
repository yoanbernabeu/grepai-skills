---
name: grepai-trace-graph
description: Build complete call graphs with GrepAI trace. Use this skill for recursive dependency analysis.
---

# GrepAI Trace Graph

This skill covers using `grepai trace graph` to build complete call graphs showing all dependencies recursively.

## When to Use This Skill

- Mapping complete function dependencies
- Understanding complex code flows
- Impact analysis for major refactoring
- Visualizing application architecture

## What is Trace Graph?

`grepai trace graph` builds a recursive dependency tree:

```
main
├── initialize
│   ├── loadConfig
│   │   └── parseYAML
│   └── connectDB
│       ├── createPool
│       └── ping
├── startServer
│   ├── registerRoutes
│   │   ├── authMiddleware
│   │   └── loggingMiddleware
│   └── listen
└── gracefulShutdown
    └── closeDB
```

## Basic Usage

```bash
grepai trace graph "FunctionName"
```

### Example

```bash
grepai trace graph "main"
```

Output:
```
🔍 Call Graph for "main"

main
├── initialize
│   ├── loadConfig
│   └── connectDB
├── startServer
│   ├── registerRoutes
│   └── listen
└── gracefulShutdown
    └── closeDB

Nodes: 9
Max depth: 3
```

## Depth Control

Limit recursion depth with `--depth`:

```bash
# Default depth (2 levels)
grepai trace graph "main"

# Deeper analysis (3 levels)
grepai trace graph "main" --depth 3

# Shallow (1 level, same as callees)
grepai trace graph "main" --depth 1

# Very deep (5 levels)
grepai trace graph "main" --depth 5
```

### Depth Examples

**--depth 1** (same as callees):
```
main
├── initialize
├── startServer
└── gracefulShutdown
```

**--depth 2** (default):
```
main
├── initialize
│   ├── loadConfig
│   └── connectDB
├── startServer
│   ├── registerRoutes
│   └── listen
└── gracefulShutdown
    └── closeDB
```

**--depth 3**:
```
main
├── initialize
│   ├── loadConfig
│   │   └── parseYAML
│   └── connectDB
│       ├── createPool
│       └── ping
├── startServer
│   ├── registerRoutes
│   │   ├── authMiddleware
│   │   └── loggingMiddleware
│   └── listen
└── gracefulShutdown
    └── closeDB
```

## JSON Output

```bash
grepai trace graph "main" --depth 2 --json
```

Output:
```json
{
  "query": "main",
  "mode": "graph",
  "depth": 2,
  "root": {
    "name": "main",
    "file": "cmd/main.go",
    "line": 10,
    "children": [
      {
        "name": "initialize",
        "file": "cmd/main.go",
        "line": 15,
        "children": [
          {
            "name": "loadConfig",
            "file": "config/config.go",
            "line": 20,
            "children": []
          },
          {
            "name": "connectDB",
            "file": "db/db.go",
            "line": 30,
            "children": []
          }
        ]
      },
      {
        "name": "startServer",
        "file": "server/server.go",
        "line": 25,
        "children": [
          {
            "name": "registerRoutes",
            "file": "server/routes.go",
            "line": 10,
            "children": []
          }
        ]
      }
    ]
  },
  "stats": {
    "nodes": 6,
    "max_depth": 2
  }
}
```

## Compact JSON

```bash
grepai trace graph "main" --depth 2 --json --compact
```

Output:
```json
{
  "q": "main",
  "d": 2,
  "r": {
    "n": "main",
    "c": [
      {"n": "initialize", "c": [{"n": "loadConfig"}, {"n": "connectDB"}]},
      {"n": "startServer", "c": [{"n": "registerRoutes"}]}
    ]
  },
  "s": {"nodes": 6, "depth": 2}
}
```

## TOON Output (v0.26.0+)

TOON format offers ~50% fewer tokens than JSON:

```bash
grepai trace graph "main" --depth 2 --toon
```

> **Note:** `--json` and `--toon` are mutually exclusive.

## Extraction Mode

Symbols and calls are always extracted with regular expressions (the "fast" extractor used by `grepai watch` and `grepai index`); output reports `"mode": "fast"`. The `--mode` flag is deprecated since grepai 1.0.0: it is still accepted for existing scripts, prints a deprecation warning and changes nothing. Do not pass `--mode precise`.

## Use Cases

### Understanding Application Flow

```bash
# Map entire application startup
grepai trace graph "main" --depth 4
```

### Impact Analysis

```bash
# What depends on this utility function?
grepai trace graph "validateInput" --depth 3

# Full impact of changing database layer
grepai trace graph "executeQuery" --depth 2
```

### Code Review

```bash
# Is this function too complex?
grepai trace graph "processOrder" --depth 5
# Many nodes = high complexity
```

### Documentation

```bash
# Generate architecture diagram data
grepai trace graph "main" --depth 3 --json > architecture.json
```

### Refactoring Planning

```bash
# What would break if we change this?
grepai trace graph "legacyAuth" --depth 3
```

## Handling Cycles

GrepAI detects and marks circular dependencies:

```
main
├── processA
│   └── processB
│       └── processA [CYCLE]
```

In JSON:
```json
{
  "name": "processA",
  "cycle": true
}
```

## Large Graphs

For very large codebases, graphs can be overwhelming:

### Limit Depth

```bash
# Start shallow
grepai trace graph "main" --depth 2
```

### Focus on Specific Areas

```bash
# Instead of main, trace specific subsystem
grepai trace graph "authMiddleware" --depth 3
```

### Filter in Post-Processing

```bash
# Get JSON and filter
grepai trace graph "main" --depth 3 --json | jq '...'
```

## Visualizing Graphs

### Export to DOT Format (Graphviz)

```bash
# Convert JSON to DOT
grepai trace graph "main" --depth 3 --json | python3 << 'EOF'
import json
import sys

data = json.load(sys.stdin)

print("digraph G {")
print("  rankdir=TB;")

def traverse(node, parent=None):
    name = node.get('name') or node.get('n')
    if parent:
        print(f'  "{parent}" -> "{name}";')
    children = node.get('children') or node.get('c') or []
    for child in children:
        traverse(child, name)

traverse(data.get('root') or data.get('r'))
print("}")
EOF
```

Then render:
```bash
dot -Tpng graph.dot -o graph.png
```

### Mermaid Diagram

```bash
grepai trace graph "main" --depth 2 --json | python3 << 'EOF'
import json
import sys

data = json.load(sys.stdin)

print("```mermaid")
print("graph TD")

def traverse(node, parent=None):
    name = node.get('name') or node.get('n')
    if parent:
        print(f"  {parent} --> {name}")
    children = node.get('children') or node.get('c') or []
    for child in children:
        traverse(child, name)

traverse(data.get('root') or data.get('r'))
print("```")
EOF
```

## Comparing Graph Sizes

Track complexity over time:

```bash
# Get node count
grepai trace graph "main" --depth 3 --json | jq '.stats.nodes'

# Compare before/after refactoring
echo "Before: $(grepai trace graph 'main' --depth 3 --json | jq '.stats.nodes') nodes"
# ... refactoring ...
echo "After: $(grepai trace graph 'main' --depth 3 --json | jq '.stats.nodes') nodes"
```

## What the Graph Contains

- Edges are real calls: property reads and writes are neither shown nor followed, and a declaration line is not an edge to itself.
- The root and same-name nodes are resolved deterministically (file path, then line), the same way in the CLI and in `grepai_trace_graph` (MCP).
- Pass a bare function name: `Class::method` is not supported.

## Common Issues

❌ **Problem:** Graph too large / timeout
✅ **Solutions:**
- Reduce depth: `--depth 2`
- Trace specific function instead of `main`

❌ **Problem:** Many cycles detected
✅ **Solution:** This indicates circular dependencies in code. Consider refactoring.

❌ **Problem:** Missing branches
✅ **Solutions:**
- Check if files are indexed
- Verify language is enabled

## Best Practices

1. **Start shallow:** Begin with `--depth 2`, increase as needed
2. **Focus analysis:** Trace specific functions, not always `main`
3. **Export for docs:** Use JSON for generating diagrams
4. **Track over time:** Monitor node count as complexity metric
5. **Investigate cycles:** Circular dependencies are code smells

## Output Format

Trace graph result:

```
🔍 Call Graph for "main"

Depth: 3
Mode: fast

main
├── initialize
│   ├── loadConfig
│   │   └── parseYAML
│   └── connectDB
│       ├── createPool
│       └── ping
├── startServer
│   ├── registerRoutes
│   │   ├── authMiddleware
│   │   └── loggingMiddleware
│   └── listen
└── gracefulShutdown
    └── closeDB

Statistics:
- Total nodes: 12
- Maximum depth reached: 3
- Cycles detected: 0

Tip: Use --json for machine-readable output
     Use --depth N to control recursion depth
```
