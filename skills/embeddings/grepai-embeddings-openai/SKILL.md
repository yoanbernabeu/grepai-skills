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
  api_key: ${OPENAI_API_KEY}
```

Set the environment variable:

```bash
export OPENAI_API_KEY="sk-..."
```

### With Parallel Processing

```yaml
embedder:
  provider: openai
  model: text-embedding-3-small
  api_key: ${OPENAI_API_KEY}
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
  api_key: ${OPENAI_API_KEY}
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

### Using .env Files

Create `.env` in your project root:

```
OPENAI_API_KEY=sk-...
```

Add to `.gitignore`:
```gitignore
.env
```

## Azure OpenAI

For Azure-hosted OpenAI:

```yaml
embedder:
  provider: openai
  model: your-deployment-name
  api_key: ${AZURE_OPENAI_API_KEY}
  endpoint: https://your-resource.openai.azure.com
```

## Security Best Practices

1. **Use environment variables:** Never hardcode API keys
2. **Add to .gitignore:** Exclude `.env` files
3. **Rotate keys:** Regularly rotate API keys
4. **Monitor usage:** Check OpenAI dashboard for unexpected usage
5. **Review code:** Ensure sensitive code isn't being indexed

## Common Issues

❌ **Problem:** `401 Unauthorized`
✅ **Solution:** Check API key is correct and environment variable is set:
```bash
echo $OPENAI_API_KEY
```

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
  api_key: ${OPENAI_API_KEY}
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
