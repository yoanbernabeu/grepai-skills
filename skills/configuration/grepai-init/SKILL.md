---
name: grepai-init
description: Initialize GrepAI in a project. Use this skill when setting up GrepAI for the first time in a codebase.
---

# GrepAI Init

This skill covers the `grepai init` command and project initialization.

## When to Use This Skill

- Setting up GrepAI in a new project
- Understanding what `grepai init` creates
- Customizing initial configuration
- Troubleshooting initialization issues

## Basic Usage

```bash
cd /path/to/your/project
grepai init
```

## What Init Creates

Running `grepai init` creates the `.grepai/` directory with:

```
.grepai/
├── config.yaml    # Configuration file
├── index.gob      # Vector index (created by watch)
└── symbols.gob    # Symbol index for trace (created by watch)
```

## Default Configuration

The generated `config.yaml`:

```yaml
version: 1

embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://localhost:11434

store:
  backend: gob

chunking:
  size: 512
  overlap: 50

watch:
  debounce_ms: 500

trace:
  mode: fast
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

ignore:
  - .git
  - .grepai
  - node_modules
  - vendor
  - target
  - __pycache__
  - dist
  - build
```

## Understanding Default Settings

### Embedder Settings

| Setting | Default | Purpose |
|---------|---------|---------|
| `provider` | `ollama` | Local embedding generation |
| `model` | `nomic-embed-text` | 768-dimension model |
| `endpoint` | `http://localhost:11434` | Ollama API URL |

### Store Settings

| Setting | Default | Purpose |
|---------|---------|---------|
| `backend` | `gob` | Local file storage |

### Chunking Settings

| Setting | Default | Purpose |
|---------|---------|---------|
| `size` | `512` | Tokens per chunk |
| `overlap` | `50` | Overlap for context |

### Watch Settings

| Setting | Default | Purpose |
|---------|---------|---------|
| `debounce_ms` | `500` | Wait time before re-indexing |

### Ignore Patterns

Default patterns exclude:
- Version control: `.git`
- GrepAI data: `.grepai`
- Dependencies: `node_modules`, `vendor`
- Build outputs: `target`, `dist`, `build`
- Cache: `__pycache__`

## Customizing After Init

Edit `.grepai/config.yaml` to customize:

### Change Embedding Provider

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  api_key: ${OPENAI_API_KEY}
```

### Change Storage Backend

```yaml
store:
  backend: postgres
  postgres:
    dsn: postgres://user:pass@localhost:5432/grepai
```

### Add Custom Ignore Patterns

```yaml
ignore:
  - .git
  - .grepai
  - node_modules
  - "*.min.js"
  - "*.bundle.js"
  - coverage/
  - .nyc_output/
```

## Init in Monorepos

For monorepos, init at the root:

```bash
cd /path/to/monorepo
grepai init
```

Or use workspaces for separate indices:

```bash
grepai workspace create my-workspace
grepai workspace add my-workspace /path/to/project1
grepai workspace add my-workspace /path/to/project2
```

## Re-Initialization

If you need to reset:

```bash
# Remove existing config
rm -rf .grepai

# Re-initialize
grepai init
```

**Warning:** This deletes your index. You'll need to re-run `grepai watch`.

## Verifying Initialization

After init, verify with:

```bash
# Check config exists
cat .grepai/config.yaml

# Check status (will show no index yet)
grepai status
```

## Common Issues

❌ **Problem:** `.grepai` already exists
✅ **Solution:** Delete it first or edit existing config:
```bash
rm -rf .grepai && grepai init
```

❌ **Problem:** Config created but Ollama not running
✅ **Solution:** Start Ollama before running `grepai watch`:
```bash
ollama serve
```

❌ **Problem:** Wrong directory initialized
✅ **Solution:** Remove `.grepai` and init in correct directory

## Best Practices

1. **Init at project root:** Where your main code lives
2. **Add `.grepai/` to `.gitignore`:** Index is machine-specific
3. **Customize ignore patterns:** Exclude generated/vendored code
4. **Review config after init:** Adjust for your stack

## Example .gitignore Addition

```gitignore
# GrepAI
.grepai/
```

## Output Format

After successful initialization:

```
✅ GrepAI Initialized

   Config: .grepai/config.yaml

   Default settings:
   - Embedder: Ollama (nomic-embed-text)
   - Storage: GOB (local file)
   - Chunking: 512 tokens, 50 overlap

   Next steps:
   1. Ensure Ollama is running: ollama serve
   2. Start indexing: grepai watch
```
