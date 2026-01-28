---
name: grepai-mcp-tools
description: Reference for all GrepAI MCP tools. Use this skill to understand available MCP tools and their parameters.
---

# GrepAI MCP Tools Reference

This skill provides a complete reference for all tools available through GrepAI's MCP server.

## When to Use This Skill

- Understanding available MCP tools
- Learning tool parameters and options
- Integrating GrepAI with AI assistants
- Debugging MCP tool usage

## Starting the MCP Server

```bash
grepai mcp-serve
```

The server exposes tools via the Model Context Protocol.

## Available Tools

### 1. grepai_search

Semantic code search using embeddings.

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | string | Yes | - | Search query describing what to find |
| `limit` | number | No | 10 | Maximum results to return |
| `compact` | boolean | No | false | Return compact JSON format |

#### Example Request

```json
{
  "tool": "grepai_search",
  "parameters": {
    "query": "user authentication middleware",
    "limit": 5,
    "compact": true
  }
}
```

#### Response (Compact)

```json
{
  "q": "user authentication middleware",
  "r": [
    {"s": 0.92, "f": "src/auth/middleware.go", "l": "15-45"},
    {"s": 0.85, "f": "src/auth/jwt.go", "l": "23-55"},
    {"s": 0.78, "f": "src/handlers/auth.go", "l": "10-40"}
  ],
  "t": 3
}
```

#### Response (Full)

```json
{
  "query": "user authentication middleware",
  "results": [
    {
      "score": 0.92,
      "file": "src/auth/middleware.go",
      "start_line": 15,
      "end_line": 45,
      "content": "func AuthMiddleware() gin.HandlerFunc {\n    ..."
    }
  ],
  "total": 3
}
```

---

### 2. grepai_trace_callers

Find all functions that call a specified symbol.

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `symbol` | string | Yes | - | Function/method name to trace |
| `compact` | boolean | No | false | Return compact JSON format |

#### Example Request

```json
{
  "tool": "grepai_trace_callers",
  "parameters": {
    "symbol": "Login",
    "compact": true
  }
}
```

#### Response (Compact)

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

#### Response (Full)

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
    }
  ]
}
```

---

### 3. grepai_trace_callees

Find all functions called by a specified symbol.

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `symbol` | string | Yes | - | Function/method name to trace |
| `compact` | boolean | No | false | Return compact JSON format |

#### Example Request

```json
{
  "tool": "grepai_trace_callees",
  "parameters": {
    "symbol": "ProcessOrder",
    "compact": true
  }
}
```

#### Response (Compact)

```json
{
  "q": "ProcessOrder",
  "m": "callees",
  "c": 4,
  "r": [
    {"f": "services/order.go", "l": 45, "fn": "validateOrder"},
    {"f": "services/order.go", "l": 48, "fn": "calculateTotal"},
    {"f": "services/order.go", "l": 51, "fn": "applyDiscount"},
    {"f": "services/order.go", "l": 55, "fn": "sendConfirmation"}
  ]
}
```

---

### 4. grepai_trace_graph

Build a complete call graph starting from a symbol.

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `symbol` | string | Yes | - | Root function for the graph |
| `depth` | number | No | 2 | Maximum recursion depth |
| `compact` | boolean | No | false | Return compact JSON format |

#### Example Request

```json
{
  "tool": "grepai_trace_graph",
  "parameters": {
    "symbol": "main",
    "depth": 3,
    "compact": true
  }
}
```

#### Response (Compact)

```json
{
  "q": "main",
  "d": 3,
  "r": {
    "n": "main",
    "c": [
      {
        "n": "initialize",
        "c": [
          {"n": "loadConfig"},
          {"n": "connectDB"}
        ]
      },
      {
        "n": "startServer",
        "c": [
          {"n": "registerRoutes"}
        ]
      }
    ]
  },
  "s": {"nodes": 6, "depth": 3}
}
```

#### Response (Full)

```json
{
  "query": "main",
  "mode": "graph",
  "depth": 3,
  "root": {
    "name": "main",
    "file": "cmd/main.go",
    "line": 10,
    "children": [
      {
        "name": "initialize",
        "file": "cmd/main.go",
        "line": 15,
        "children": [...]
      }
    ]
  },
  "stats": {
    "nodes": 6,
    "max_depth": 3
  }
}
```

---

### 5. grepai_index_status

Check the health and status of the code index.

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `verbose` | boolean | No | false | Include detailed information |

#### Example Request

```json
{
  "tool": "grepai_index_status",
  "parameters": {
    "verbose": true
  }
}
```

#### Response

```json
{
  "status": "healthy",
  "project": "/path/to/project",
  "embedder": {
    "provider": "ollama",
    "model": "nomic-embed-text",
    "status": "connected"
  },
  "store": {
    "backend": "gob",
    "location": ".grepai/index.gob"
  },
  "index": {
    "files": 245,
    "chunks": 1234,
    "last_updated": "2025-01-28T10:30:00Z"
  },
  "daemon": {
    "running": true,
    "pid": 12345
  }
}
```

## Compact Format Reference

When `compact: true`, responses use abbreviated keys:

| Full Key | Compact Key | Description |
|----------|-------------|-------------|
| `query` | `q` | Search query or symbol |
| `results` | `r` | Results array |
| `total` | `t` | Total count |
| `count` | `c` | Count |
| `score` | `s` | Similarity score |
| `file` | `f` | File path |
| `line` | `l` | Line number(s) |
| `mode` | `m` | Trace mode |
| `depth` | `d` | Graph depth |
| `name` | `n` | Node name |
| `children` | `c` | Child nodes |
| `stats` | `s` | Statistics |
| `function` | `fn` | Function name |

## Token Efficiency

Compact mode reduces tokens significantly:

| Response Type | Full | Compact | Savings |
|---------------|------|---------|---------|
| Search (5 results) | ~800 | ~150 | 81% |
| Trace callers (10) | ~600 | ~120 | 80% |
| Trace graph (depth 3) | ~1200 | ~250 | 79% |

## Error Responses

### Index Not Found

```json
{
  "error": "Index not found. Run 'grepai watch' first.",
  "code": "INDEX_NOT_FOUND"
}
```

### Embedder Connection Failed

```json
{
  "error": "Cannot connect to embedding provider. Is Ollama running?",
  "code": "EMBEDDER_UNAVAILABLE"
}
```

### Symbol Not Found

```json
{
  "error": "Symbol 'FunctionName' not found in index.",
  "code": "SYMBOL_NOT_FOUND"
}
```

### Invalid Parameters

```json
{
  "error": "Parameter 'query' is required.",
  "code": "INVALID_PARAMETERS"
}
```

## Best Practices for AI Integration

1. **Use compact mode:** Reduces token usage by ~80%
2. **Limit results:** Request only what you need
3. **Check status first:** Use `grepai_index_status` before searches
4. **Handle errors:** Check for error responses
5. **Combine tools:** Search + trace for full understanding

## MCP Protocol Details

### Request Format

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "grepai_search",
    "arguments": {
      "query": "authentication",
      "limit": 5,
      "compact": true
    }
  }
}
```

### Response Format

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"q\":\"authentication\",\"r\":[...],\"t\":5}"
      }
    ]
  }
}
```

## Output Format

MCP tools reference summary:

```
📚 GrepAI MCP Tools Reference

Tools available:

1. grepai_search
   - Semantic code search
   - Params: query*, limit, compact

2. grepai_trace_callers
   - Find function callers
   - Params: symbol*, compact

3. grepai_trace_callees
   - Find function callees
   - Params: symbol*, compact

4. grepai_trace_graph
   - Build call graph
   - Params: symbol*, depth, compact

5. grepai_index_status
   - Check index health
   - Params: verbose

* = required parameter

Compact mode: ~80% token reduction
```
