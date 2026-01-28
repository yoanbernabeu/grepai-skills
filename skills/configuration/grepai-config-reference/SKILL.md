---
name: grepai-config-reference
description: Complete configuration reference for GrepAI. Use this skill when you need to understand all available configuration options.
---

# GrepAI Configuration Reference

This skill provides a complete reference for all GrepAI configuration options in `.grepai/config.yaml`.

## When to Use This Skill

- Understanding all available configuration options
- Optimizing GrepAI for your specific use case
- Troubleshooting configuration issues
- Setting up advanced configurations

## Configuration File Location

```
/your/project/.grepai/config.yaml
```

## Complete Configuration Schema

```yaml
version: 1

# ═══════════════════════════════════════════════════════════════
# EMBEDDER CONFIGURATION
# Converts code text into vector embeddings
# ═══════════════════════════════════════════════════════════════
embedder:
  # Provider: ollama | openai | lmstudio
  provider: ollama

  # Model name (depends on provider)
  # Ollama: nomic-embed-text, bge-m3, mxbai-embed-large
  # OpenAI: text-embedding-3-small, text-embedding-3-large
  # LM Studio: nomic-embed-text-v1.5, bge-small-en-v1.5
  model: nomic-embed-text

  # API endpoint URL
  # Ollama default: http://localhost:11434
  # LM Studio default: http://localhost:1234
  # OpenAI: uses official API
  endpoint: http://localhost:11434

  # Vector dimensions (auto-detected if omitted)
  # nomic-embed-text: 768
  # text-embedding-3-small: 1536
  # text-embedding-3-large: 3072
  dimensions: 768

  # API key (for OpenAI, supports env vars)
  api_key: ${OPENAI_API_KEY}

  # Parallel requests (OpenAI only, for speed)
  parallelism: 4

# ═══════════════════════════════════════════════════════════════
# STORE CONFIGURATION
# Where vector embeddings are stored
# ═══════════════════════════════════════════════════════════════
store:
  # Backend: gob | postgres | qdrant
  backend: gob

  # PostgreSQL configuration (when backend: postgres)
  postgres:
    dsn: postgres://user:password@localhost:5432/grepai

  # Qdrant configuration (when backend: qdrant)
  qdrant:
    endpoint: localhost
    port: 6334
    use_tls: false
    api_key: your-qdrant-api-key  # Optional

# ═══════════════════════════════════════════════════════════════
# CHUNKING CONFIGURATION
# How code files are split for embedding
# ═══════════════════════════════════════════════════════════════
chunking:
  # Tokens per chunk (smaller = more precise, larger = more context)
  # Recommended: 256-1024
  size: 512

  # Overlap between chunks (preserves context at boundaries)
  # Recommended: 10-20% of size
  overlap: 50

# ═══════════════════════════════════════════════════════════════
# WATCH CONFIGURATION
# File watching daemon settings
# ═══════════════════════════════════════════════════════════════
watch:
  # Debounce delay in milliseconds
  # Groups rapid file changes together
  debounce_ms: 500

# ═══════════════════════════════════════════════════════════════
# TRACE CONFIGURATION
# Call graph analysis settings
# ═══════════════════════════════════════════════════════════════
trace:
  # Extraction mode: fast | precise
  # fast: Uses regex, no dependencies, faster
  # precise: Uses tree-sitter AST parsing, more accurate
  mode: fast

  # Languages to analyze for call graphs
  enabled_languages:
    - .go
    - .js
    - .ts
    - .jsx
    - .tsx
    - .py
    - .php
    - .c
    - .h
    - .cpp
    - .hpp
    - .cc
    - .cxx
    - .rs
    - .zig
    - .cs
    - .pas
    - .dpr

  # Patterns to exclude from trace analysis
  exclude_patterns:
    - "*_test.go"
    - "*.spec.ts"
    - "*.test.js"

# ═══════════════════════════════════════════════════════════════
# SEARCH CONFIGURATION
# Search result scoring and ranking
# ═══════════════════════════════════════════════════════════════
search:
  # Score boosting configuration
  boost:
    enabled: true

    # Reduce scores for certain paths
    penalties:
      - pattern: /tests/
        factor: 0.5
      - pattern: _test.
        factor: 0.5
      - pattern: .spec.
        factor: 0.5
      - pattern: /docs/
        factor: 0.6
      - pattern: /vendor/
        factor: 0.3
      - pattern: /node_modules/
        factor: 0.3

    # Increase scores for certain paths
    bonuses:
      - pattern: /src/
        factor: 1.1
      - pattern: /lib/
        factor: 1.1
      - pattern: /core/
        factor: 1.2
      - pattern: /app/
        factor: 1.1

  # Hybrid search (vector + keyword)
  hybrid:
    enabled: false
    k: 60  # BM25 parameter

# ═══════════════════════════════════════════════════════════════
# IGNORE CONFIGURATION
# Files and directories to exclude from indexing
# ═══════════════════════════════════════════════════════════════
ignore:
  # Directories
  - .git
  - .grepai
  - .svn
  - .hg
  - node_modules
  - vendor
  - target
  - __pycache__
  - .pytest_cache
  - dist
  - build
  - out
  - .next
  - .nuxt

  # Files
  - "*.min.js"
  - "*.min.css"
  - "*.bundle.js"
  - "*.map"
  - "*.lock"
  - package-lock.json
  - yarn.lock
  - pnpm-lock.yaml
  - go.sum

  # Generated
  - "*.generated.*"
  - "*.pb.go"
  - "*.d.ts"
```

## Configuration by Use Case

### Small Personal Project

```yaml
version: 1
embedder:
  provider: ollama
  model: nomic-embed-text
store:
  backend: gob
chunking:
  size: 512
  overlap: 50
```

### Large Codebase

```yaml
version: 1
embedder:
  provider: ollama
  model: bge-m3  # Larger model
  parallelism: 4
store:
  backend: postgres  # Scalable storage
  postgres:
    dsn: postgres://user:pass@localhost:5432/grepai
chunking:
  size: 768  # Larger chunks
  overlap: 100
```

### Team Environment

```yaml
version: 1
embedder:
  provider: openai
  model: text-embedding-3-small
  api_key: ${OPENAI_API_KEY}
  parallelism: 8
store:
  backend: qdrant
  qdrant:
    endpoint: qdrant.internal.company.com
    port: 6334
    use_tls: true
```

### Maximum Privacy

```yaml
version: 1
embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://localhost:11434
store:
  backend: gob  # Local file only
```

## Environment Variables

GrepAI supports environment variable substitution:

```yaml
embedder:
  api_key: ${OPENAI_API_KEY}

store:
  postgres:
    dsn: ${DATABASE_URL}
```

Set in your shell:

```bash
export OPENAI_API_KEY="sk-..."
export DATABASE_URL="postgres://..."
```

## Validating Configuration

Check your config is valid:

```bash
grepai status
```

If there are config errors, they'll be displayed.

## Configuration Precedence

1. `.grepai/config.yaml` in current directory
2. Workspace configuration (if using workspaces)
3. Default values

## Best Practices

1. **Start simple:** Use defaults, optimize later
2. **Match chunking to code style:** Larger chunks for verbose code
3. **Use boosting:** Penalize test/vendor, boost src/lib
4. **Secure API keys:** Use environment variables, never commit
5. **Exclude noise:** Ignore generated files, dependencies

## Output Format

Valid configuration status:

```
✅ GrepAI Configuration Valid

   Embedder: ollama (nomic-embed-text)
   Storage: gob (.grepai/index.gob)
   Chunking: 512 tokens, 50 overlap
   Trace mode: fast
   Languages: 18 enabled
   Ignore patterns: 12 configured
   Boosting: enabled
```
