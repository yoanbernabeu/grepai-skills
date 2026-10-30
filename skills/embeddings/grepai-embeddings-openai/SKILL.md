---
name: grepai-embeddings-openai
description: Configure OpenAI as embedding provider for GrepAI. Use this skill for high-quality cloud embeddings.
---

# GrepAI Embeddings with OpenAI

This skill covers using OpenAI's embedding API with GrepAI for high-quality, cloud-based embeddings.

## When to Use This Skill

- Need highest quality embeddings
- Team environment with shared infrastructure
- Don't want to manage local embedding server
- Willing to trade privacy for quality/convenience

## Considerations

| Aspect | Details |
|--------|---------|
| ✅ **Quality** | State-of-the-art embeddings |
| ✅ **Speed** | Fast, no local compute needed |
| ✅ **Scalability** | Handles any codebase size |
| ⚠️ **Privacy** | Code sent to OpenAI servers |
| ⚠️ **Cost** | Pay per token |
| ⚠️ **Internet** | Requires connection |

## Prerequisites

1. OpenAI API key
2. Billing enabled on OpenAI account

Get your API key at: https://platform.openai.com/api-keys

## Configuration

### Basic Configuration

```yaml
# .grepai/config.yaml
embedder:
  provider: openai
  model: text-embedding-3-small
```

Set the environment variable:

```bash
export OPENAI_API_KEY="sk-..."
```

With no `api_key` and no `endpoint`, grepai uses the official endpoint (`https://api.openai.com/v1`) and reads `OPENAI_API_KEY` from the environment.

> **Note:** grepai does not expand variables in `config.yaml`. `api_key: ${OPENAI_API_KEY}` is sent as the literal string `${OPENAI_API_KEY}`; leave `api_key` out to use the environment variable.

### With Parallel Processing

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  parallelism: 8  # Concurrent requests for speed
```

### Direct API Key (Not Recommended)

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  api_key: sk-your-api-key-here  # Avoid committing secrets!
```

**Warning:** Never commit API keys to version control.

## Available Models

### text-embedding-3-small (Recommended)

| Property | Value |
|----------|-------|
| Dimensions | 1536 |
| Price | $0.00002 / 1K tokens |
| Quality | Very high |
| Speed | Fast |

**Best for:** Most use cases, good balance of cost/quality.

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
```

### text-embedding-3-large

| Property | Value |
|----------|-------|
| Dimensions | 3072 |
| Price | $0.00013 / 1K tokens |
| Quality | Highest |
| Speed | Fast |

**Best for:** Maximum accuracy, cost not a concern.

```yaml
embedder:
  provider: openai
  model: text-embedding-3-large
  dimensions: 3072
```

### Dimension Reduction

You can reduce dimensions to save storage:

```yaml
embedder:
  provider: openai
  model: text-embedding-3-large
  dimensions: 1024  # Reduced from 3072
```

## Model Comparison

| Model | Dimensions | Cost/1K tokens | Quality |
|-------|------------|----------------|---------|
| `text-embedding-3-small` | 1536 | $0.00002 | ⭐⭐⭐⭐ |
| `text-embedding-3-large` | 3072 | $0.00013 | ⭐⭐⭐⭐⭐ |

## Cost Estimation

Approximate costs per 1000 source files:

| Codebase Size | Chunks | Small Model | Large Model |
|---------------|--------|-------------|-------------|
| Small (100 files) | ~500 | $0.01 | $0.06 |
| Medium (1000 files) | ~5,000 | $0.10 | $0.65 |
| Large (10000 files) | ~50,000 | $1.00 | $6.50 |

**Note:** Costs are one-time for initial indexing. Updates only re-embed changed files.

## Optimizing for Speed

### Parallel Requests

GrepAI v0.24.0+ supports adaptive rate limiting and parallel requests:

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  parallelism: 8  # Adjust based on your rate limit tier
```

Parallelism recommendations:
- **Tier 1 (Free):** 1-2
- **Tier 2:** 4-8
- **Tier 3+:** 8-16

### Batching

GrepAI automatically batches chunks for efficient API usage.

## Rate Limits

OpenAI has rate limits based on your account tier:

| Tier | RPM | TPM |
|------|-----|-----|
| Free | 3 | 150,000 |
| Tier 1 | 500 | 1,000,000 |
| Tier 2 | 5,000 | 5,000,000 |

GrepAI handles rate limiting automatically with adaptive backoff.

## Environment Variables

### Setting the API Key

**macOS/Linux:**
```bash
# In ~/.bashrc, ~/.zshrc, or ~/.profile
export OPENAI_API_KEY="sk-..."
```

**Windows (PowerShell):**
```powershell
$env:OPENAI_API_KEY = "sk-..."
# Or permanently
[System.Environment]::SetEnvironmentVariable('OPENAI_API_KEY', 'sk-...', 'User')
```

grepai does not read `.env` files: the variable must be exported in the shell (or in the `env` block of your MCP client config) that starts grepai.

### Environment Keys Only Go to the Official Endpoint

Since grepai v1.2.0, a key read from the environment is sent only to `https://api.openai.com`. With any other `endpoint` (Azure, LiteLLM, vLLM, a proxy…), set `embedder.api_key` in `.grepai/config.yaml` (or `~/.grepai/workspace.yaml` for a workspace). If `api_key` is empty while `OPENAI_API_KEY` is exported, grepai stops with an error naming the variable and the endpoint, instead of sending the key.

This protects you from a `.grepai/config.yaml` committed in a cloned repository that points `endpoint` at a server controlled by someone else.

## Azure OpenAI

For Azure-hosted OpenAI (or any OpenAI-compatible API):

```yaml
embedder:
  provider: openai
  model: your-deployment-name
  endpoint: https://your-resource.openai.azure.com/openai/v1
  api_key: your-azure-key  # required: OPENAI_API_KEY is never sent to a custom endpoint
```

grepai sends `POST {endpoint}/embeddings` with an `Authorization: Bearer` header. Keep a config holding a key out of version control.

### Local OpenAI-Compatible Servers (vLLM, TEI…)

The `openai` provider always requires a non-empty key, so a server that needs no authentication still needs a placeholder. Without one, grepai stops with `OpenAI API key not set` (no variable exported) or with the custom endpoint error (`OPENAI_API_KEY` exported):

```yaml
embedder:
  provider: openai
  model: your-model
  endpoint: http://localhost:8000/v1
  api_key: unused
```

## Security Best Practices

1. **Use environment variables with the official endpoint:** Never hardcode API keys for `api.openai.com`
2. **Custom endpoints:** The key must be in `api_key`; never commit that config
3. **Rotate keys:** Regularly rotate API keys
4. **Monitor usage:** Check OpenAI dashboard for unexpected usage
5. **Review code:** Ensure sensitive code isn't being indexed

## Common Issues

❌ **Problem:** `401 Unauthorized`
✅ **Solution:** Check API key is correct and environment variable is set:
```bash
echo $OPENAI_API_KEY
```

❌ **Problem:** `API keys from the environment (OPENAI_API_KEY) are only sent to the official endpoint ... not to the custom endpoint ...`
✅ **Solution:** You use a custom `endpoint`. Set `embedder.api_key` in the config (`api_key: unused` for a server without authentication).

❌ **Problem:** `429 Rate limit exceeded`
✅ **Solution:** Reduce parallelism or upgrade OpenAI tier:
```yaml
embedder:
  parallelism: 2  # Lower value
```

❌ **Problem:** High costs
✅ **Solutions:**
- Use `text-embedding-3-small` instead of large
- Reduce dimension size
- Add more ignore patterns to reduce indexed files

❌ **Problem:** Slow indexing
✅ **Solution:** Increase parallelism:
```yaml
embedder:
  parallelism: 8
```

❌ **Problem:** Privacy concerns
✅ **Solution:** Use Ollama for local embeddings instead

## Migrating from Ollama to OpenAI

1. Update configuration:
```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
```

2. Delete existing index:
```bash
rm .grepai/index.gob
```

3. Re-index:
```bash
grepai watch
```

**Important:** You cannot mix embeddings from different models/providers.

## Output Format

Successful OpenAI configuration:

```
✅ OpenAI Embedding Provider Configured

   Provider: OpenAI
   Model: text-embedding-3-small
   Dimensions: 1536
   Parallelism: 4
   API Key: sk-...xxxx (from environment)

   Estimated cost for this codebase:
   - Files: 245
   - Chunks: ~1,200
   - Cost: ~$0.02

   Note: Code will be sent to OpenAI servers.
```
