---
name: grepai-troubleshooting
description: Troubleshooting guide for GrepAI. Use this skill to diagnose and fix common issues.
---

# GrepAI Troubleshooting

This skill provides solutions for common GrepAI issues and diagnostic procedures.

## When to Use This Skill

- GrepAI not working as expected
- Search returning poor results
- Index not updating
- Connection or configuration errors

## Quick Diagnostics

Run these commands to understand your setup:

```bash
# Check GrepAI version
grepai version

# Check project status
grepai status

# Check Ollama (if using)
curl http://localhost:11434/api/tags

# Check config
cat .grepai/config.yaml
```

## Common Issues

---

### Issue: "Index not found"

**Symptom:**
```
Error: Index not found. Run 'grepai watch' first.
```

**Cause:** No index has been created for this project.

**Solution:**
```bash
# Initialize if needed
grepai init

# Create the index
grepai watch
```

---

### Issue: "Cannot connect to embedding provider"

**Symptom:**
```
Error: Cannot connect to Ollama at http://localhost:11434
```

**Causes:**
1. Ollama not running
2. Wrong endpoint configured
3. Firewall blocking connection

**Solutions:**

1. Start Ollama:
```bash
ollama serve
```

2. Check endpoint in config:
```yaml
embedder:
  endpoint: http://localhost:11434  # Verify this
```

3. Test connection:
```bash
curl http://localhost:11434/api/tags
```

---

### Issue: "Model not found"

**Symptom:**
```
Error: Model 'nomic-embed-text' not found
```

**Cause:** The embedding model hasn't been downloaded.

**Solution:**
```bash
# Download the model
ollama pull nomic-embed-text

# Verify
ollama list
```

---

### Issue: Search returns no results

**Symptom:** Searches return empty or very few results.

**Causes:**
1. Index is empty
2. Files are being ignored
3. Query too specific

**Solutions:**

1. Check index status:
```bash
grepai status
# Should show files > 0 and chunks > 0
```

2. Verify files are being indexed:
```bash
# Check ignore patterns in config
cat .grepai/config.yaml | grep -A 20 "ignore:"
```

3. Try broader query:
```bash
grepai search "function"  # Very broad test
```

---

### Issue: Search returns irrelevant results

**Symptom:** Results don't match what you're looking for.

**Causes:**
1. Query too vague
2. Boosting not configured
3. Wrong content indexed

**Solutions:**

1. Improve query (see `grepai-search-tips` skill):
```bash
# Bad
grepai search "auth"

# Good
grepai search "user authentication middleware"
```

2. Configure boosting to penalize tests:
```yaml
search:
  boost:
    enabled: true
    penalties:
      - pattern: /tests/
        factor: 0.5
```

3. Check what's indexed:
```bash
grepai status
```

---

### Issue: Index is outdated

**Symptom:** Recent file changes aren't appearing in search results.

**Causes:**
1. Watch daemon not running
2. Debounce delay
3. File not in indexed extensions

**Solutions:**

1. Check daemon status:
```bash
grepai watch --status
```

2. Restart daemon:
```bash
grepai watch --stop
grepai watch --background
```

3. Force re-index:
```bash
rm .grepai/index.gob
grepai watch
```

---

### Issue: "Config not found"

**Symptom:**
```
Error: Config file not found at .grepai/config.yaml
```

**Cause:** GrepAI not initialized in this directory.

**Solution:**
```bash
grepai init
```

---

### Issue: Slow indexing

**Symptom:** Initial indexing takes very long.

**Causes:**
1. Large codebase
2. Slow embedding provider
3. Not enough ignore patterns

**Solutions:**

1. Add ignore patterns:
```yaml
ignore:
  - node_modules
  - vendor
  - dist
  - build
  - "*.min.js"
```

2. Use faster model:
```yaml
embedder:
  model: nomic-embed-text  # Smaller, faster
```

3. Use OpenAI for speed (if privacy allows):
```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  parallelism: 8
```

---

### Issue: Slow searches

**Symptom:** Search queries take several seconds.

**Causes:**
1. Very large index
2. GOB storage on large codebase
3. Embedding provider slow

**Solutions:**

1. Check index size:
```bash
ls -lh .grepai/index.gob
```

2. For large indices, use Qdrant:
```yaml
store:
  backend: qdrant
```

3. Limit results:
```bash
grepai search "query" --limit 5
```

---

### Issue: Trace not finding symbols

**Symptom:** `grepai trace callers` returns no results.

**Causes:**
1. Function name spelled wrong
2. Language not enabled for trace
3. Symbols index out of date

**Solutions:**

1. Check exact function name (case-sensitive)

2. Enable language in config:
```yaml
trace:
  enabled_languages:
    - .go
    - .js
    - .ts
```

3. Re-build symbol index:
```bash
rm .grepai/symbols.gob
grepai watch
```

---

### Issue: MCP not working

**Symptom:** AI assistant can't use GrepAI tools.

**Causes:**
1. MCP config incorrect
2. GrepAI not in PATH
3. Working directory wrong

**Solutions:**

1. Test MCP server manually:
```bash
grepai mcp-serve
```

2. Check GrepAI is in PATH:
```bash
which grepai
```

3. Verify MCP config:
```bash
# Claude Code
cat ~/.claude/mcp.json

# Cursor
cat .cursor/mcp.json
```

---

### Issue: Out of memory

**Symptom:** GrepAI crashes or system becomes slow.

**Causes:**
1. Large embedding model
2. Very large index in GOB format
3. Too many parallel requests

**Solutions:**

1. Use smaller model:
```yaml
embedder:
  model: nomic-embed-text  # Smaller
```

2. Use PostgreSQL or Qdrant instead of GOB

3. Reduce parallelism:
```yaml
embedder:
  parallelism: 2
```

---

### Issue: API key errors (OpenAI)

**Symptom:**
```
Error: 401 Unauthorized - Invalid API key
```

**Solutions:**

1. Check environment variable:
```bash
echo $OPENAI_API_KEY
```

2. Ensure variable is exported:
```bash
export OPENAI_API_KEY="sk-..."
```

3. Check key format in config. grepai does not substitute `${VAR}`: `api_key: ${OPENAI_API_KEY}` is sent literally. Remove the line to use the environment variable:
```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  # no api_key: OPENAI_API_KEY is read from the environment
```

---

### Issue: "API keys from the environment ... are only sent to the official endpoint"

**Symptom:**
```
API keys from the environment (OPENAI_API_KEY) are only sent to the official endpoint https://api.openai.com/v1, not to the custom endpoint https://...: set embedder.api_key in the grepai config to use a key with this endpoint
```

**Cause:** `embedder.endpoint` is not the provider's official host over HTTPS (`api.openai.com`, `openrouter.ai`, `api.synthetic.new`, `router.requesty.ai`), and `embedder.api_key` is empty. grepai never sends an environment key (`OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `SYNTHETIC_API_KEY`, `REQUESTY_API_KEY`) to another endpoint, because a `.grepai/config.yaml` committed in a cloned repository could redirect it.

**Solutions:**

1. Azure, LiteLLM, vLLM, a proxy: put the key in the config (`.grepai/config.yaml` or `~/.grepai/workspace.yaml`), and keep that file out of version control:
```yaml
embedder:
  provider: openai
  endpoint: https://your-resource.openai.azure.com/openai/v1
  api_key: your-key
```

2. Local server without authentication (through the `openai` provider): the provider always needs a non-empty key, so use a placeholder:
```yaml
embedder:
  api_key: unused
```

3. Using the official API: remove `endpoint` (empty keeps the official endpoint).

`ollama` and `lmstudio` providers use no key and are not affected.

---

### Issue: "framework_processing.node_path ... use \"node\" or an absolute path outside the project"

**Symptom:** A warning at `index` or `watch` time (only while Vue processing is enabled), for example:
```
Warning: framework_processing.node_path "./node_modules/.bin/node" is a relative path: use "node" or an absolute path outside the project; using "node" instead
```

**Cause:** `node_path` only accepts a Node.js command name resolved through `PATH` (`node`, `nodejs`, `node20`, `node.exe`) or a local absolute path outside the project and its git repository. A relative path, a path inside the project or its repository (also through symlinks), a missing path, a `/proc` or `/dev` path or a network path is ignored, and grepai uses `node` from `PATH`.

**Solution:** Remove `node_path` (default `node`), or set an absolute path outside the project:
```yaml
framework_processing:
  node_path: /usr/local/bin/node
```

**Untrusted repositories:** compiling `.vue` files loads `@vue/compiler-sfc` from the project's `node_modules`. Only run grepai on repositories you trust, or disable Vue processing:
```yaml
framework_processing:
  frameworks:
    vue:
      enabled: false
```

---

### Issue: "watcher is already running"

**Symptom:**
```
watcher is already running in background (PID 12345)
Use 'grepai watch --stop' to stop it
```

If two starts race, the second one fails with `another grepai watch process is starting (lock held)` (`another grepai worktree watch process is starting (lock held)` in a linked worktree).

**Cause:** Only one watch daemon can run per project or worktree: the PID lock is held for the daemon's whole lifetime.

**Solution:** Use the running daemon, or stop it first:
```bash
grepai watch --status
grepai watch --stop
```

An empty `*.pid.lock` file stays in the log directory after a stop. This is normal; do not delete it while a daemon runs.

---

### Issue: Qdrant hybrid search returns a single result

**Symptom:** With `store.backend: qdrant` and `search.hybrid.enabled: true`, every search returns only one result.

**Cause:** Bug present since v0.18.0, fixed in grepai v1.2.0.

**Solution:** Upgrade grepai (`grepai update`). No re-index is needed. Results with hybrid search disabled are unchanged.

---

### Issue: Empty RPG links in a workspace, or a single RPG node with Qdrant

**Symptom:** In workspace mode, RPG symbols are not linked to chunks; or with Qdrant and RPG, all chunks end up in one node.

**Solution:** Fixed in grepai v1.2.0. Upgrade, then run `grepai index` (or let `grepai watch` run): the links are repaired on the next indexing, without re-embedding.

## Diagnostic Commands

### Full System Check

```bash
#!/bin/bash
echo "=== GrepAI Diagnostics ==="

echo -e "\n1. Version:"
grepai version

echo -e "\n2. Status:"
grepai status

echo -e "\n3. Config:"
cat .grepai/config.yaml 2>/dev/null || echo "No config found"

echo -e "\n4. Index files:"
ls -la .grepai/ 2>/dev/null || echo "No .grepai directory"

echo -e "\n5. Ollama (if using):"
curl -s http://localhost:11434/api/tags | head -5 || echo "Ollama not responding"

echo -e "\n6. Daemon:"
grepai watch --status 2>/dev/null || echo "Daemon not running"
```

### Reset Everything

If all else fails, complete reset:

```bash
# Remove all GrepAI data
rm -rf .grepai

# Re-initialize
grepai init

# Start fresh index
grepai watch
```

## Getting Help

If issues persist:

1. Check GrepAI documentation: https://yoanbernabeu.github.io/grepai/
2. Search issues: https://github.com/yoanbernabeu/grepai/issues
3. Create new issue with:
   - GrepAI version (`grepai version`)
   - OS and architecture
   - Config file (remove secrets)
   - Error message
   - Steps to reproduce

## Output Format

Diagnostic summary:

```
🔍 GrepAI Diagnostics

Version: 0.24.0
Project: /path/to/project

✅ Config: Found (.grepai/config.yaml)
✅ Index: 245 files, 1,234 chunks
✅ Embedder: Ollama (connected)
✅ Daemon: Running (PID 12345)
❌ Issue: [Description if any]

Recommended actions:
1. [Action item]
2. [Action item]
```
