---
name: grepai-embeddings-ollama
description: Configure Ollama as embedding provider for GrepAI. Use this skill for local, private embedding generation.
---

# GrepAI Embeddings with Ollama

This skill covers using Ollama as the embedding provider for GrepAI, enabling 100% private, local code search.

## When to Use This Skill

- Setting up private, local embeddings
- Choosing the right Ollama model
- Optimizing Ollama performance
- Troubleshooting Ollama connection issues

## Why Ollama?

| Advantage | Description |
|-----------|-------------|
| 🔒 **Privacy** | Code never leaves your machine |
| 💰 **Free** | No API costs or usage limits |
| ⚡ **Speed** | No network latency |
| 🔌 **Offline** | Works without internet |
| 🔧 **Control** | Choose your model |

## Prerequisites

1. Ollama installed and running
2. An embedding model downloaded

```bash
# Install Ollama
brew install ollama  # macOS
# or
curl -fsSL https://ollama.com/install.sh | sh  # Linux

# Start Ollama
ollama serve

# Download model
ollama pull nomic-embed-text
```

## Configuration

### Basic Configuration

```yaml
# .grepai/config.yaml
embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://localhost:11434
```

### With Custom Endpoint

```yaml
embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://192.168.1.100:11434  # Remote Ollama server
```

### With Explicit Dimensions

```yaml
embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://localhost:11434
  dimensions: 768  # Usually auto-detected
```

## Available Models

### Recommended: nomic-embed-text

```bash
ollama pull nomic-embed-text
```

| Property | Value |
|----------|-------|
| Dimensions | 768 |
| Size | ~274 MB |
| Speed | Fast |
| Quality | Excellent for code |
| Language | English-optimized |

**Configuration:**
```yaml
embedder:
  provider: ollama
  model: nomic-embed-text
```

### Multilingual: nomic-embed-text-v2-moe

```bash
ollama pull nomic-embed-text-v2-moe
```

| Property | Value |
|----------|-------|
| Dimensions | 768 |
| Size | ~500 MB |
| Speed | Medium |
| Quality | Excellent |
| Language | Multilingual |

Best for codebases with non-English comments/documentation.

**Configuration:**
```yaml
embedder:
  provider: ollama
  model: nomic-embed-text-v2-moe
```

### High Quality: bge-m3

```bash
ollama pull bge-m3
```

| Property | Value |
|----------|-------|
| Dimensions | 1024 |
| Size | ~1.2 GB |
| Speed | Slower |
| Quality | Very high |
| Language | Multilingual |

Best for large, complex codebases where accuracy is critical.

**Configuration:**
```yaml
embedder:
  provider: ollama
  model: bge-m3
  dimensions: 1024
```

### Maximum Quality: mxbai-embed-large

```bash
ollama pull mxbai-embed-large
```

| Property | Value |
|----------|-------|
| Dimensions | 1024 |
| Size | ~670 MB |
| Speed | Medium |
| Quality | Highest |
| Language | English |

**Configuration:**
```yaml
embedder:
  provider: ollama
  model: mxbai-embed-large
  dimensions: 1024
```

## Model Comparison

| Model | Dims | Size | Speed | Quality | Use Case |
|-------|------|------|-------|---------|----------|
| `nomic-embed-text` | 768 | 274MB | ⚡⚡⚡ | ⭐⭐⭐ | General use |
| `nomic-embed-text-v2-moe` | 768 | 500MB | ⚡⚡ | ⭐⭐⭐⭐ | Multilingual |
| `bge-m3` | 1024 | 1.2GB | ⚡ | ⭐⭐⭐⭐⭐ | Large codebases |
| `mxbai-embed-large` | 1024 | 670MB | ⚡⚡ | ⭐⭐⭐⭐⭐ | Maximum accuracy |

## Performance Optimization

### Memory Management

Models load into RAM. Ensure sufficient memory:

| Model | RAM Required |
|-------|--------------|
| `nomic-embed-text` | ~500 MB |
| `nomic-embed-text-v2-moe` | ~800 MB |
| `bge-m3` | ~1.5 GB |
| `mxbai-embed-large` | ~1 GB |

### GPU Acceleration

Ollama automatically uses:
- **macOS:** Metal (Apple Silicon)
- **Linux/Windows:** CUDA (NVIDIA GPUs)

Check GPU usage:
```bash
ollama ps
```

### Keeping Model Loaded

By default, Ollama unloads models after 5 minutes of inactivity. Keep loaded:

```bash
# Keep model loaded indefinitely
curl http://localhost:11434/api/generate -d '{
  "model": "nomic-embed-text",
  "keep_alive": -1
}'
```

## Verifying Connection

### Check Ollama is Running

```bash
curl http://localhost:11434/api/tags
```

### List Available Models

```bash
ollama list
```

### Test Embedding

```bash
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "function authenticate(user, password)"
}'
```

## Running Ollama as a Service

### macOS (launchd)

Ollama app runs automatically on login.

### Linux (systemd)

```bash
# Enable service
sudo systemctl enable ollama

# Start service
sudo systemctl start ollama

# Check status
sudo systemctl status ollama
```

### Manual Background

```bash
nohup ollama serve > /dev/null 2>&1 &
```

## Remote Ollama Server

Run Ollama on a powerful server and connect remotely:

### On the Server

```bash
# Allow remote connections
OLLAMA_HOST=0.0.0.0 ollama serve
```

### On the Client

```yaml
# .grepai/config.yaml
embedder:
  provider: ollama
  model: nomic-embed-text
  endpoint: http://server-ip:11434
```

## Common Issues

❌ **Problem:** Connection refused
✅ **Solution:**
```bash
# Start Ollama
ollama serve
```

❌ **Problem:** Model not found
✅ **Solution:**
```bash
# Pull the model
ollama pull nomic-embed-text
```

❌ **Problem:** Slow embedding generation
✅ **Solutions:**
- Use a smaller model (`nomic-embed-text`)
- Ensure GPU is being used (`ollama ps`)
- Close memory-intensive applications
- Consider a remote server with better hardware

❌ **Problem:** Out of memory
✅ **Solutions:**
- Use a smaller model
- Close other applications
- Upgrade RAM
- Use remote Ollama server

❌ **Problem:** Embeddings differ after model update
✅ **Solution:** Re-index after model updates:
```bash
rm .grepai/index.gob
grepai watch
```

## Best Practices

1. **Start with `nomic-embed-text`:** Best balance of speed/quality
2. **Keep Ollama running:** Background service recommended
3. **Match dimensions:** Don't mix models with different dimensions
4. **Re-index on model change:** Delete index and re-run watch
5. **Monitor memory:** Embedding models use significant RAM

## Output Format

Successful Ollama configuration:

```
✅ Ollama Embedding Provider Configured

   Provider: Ollama
   Model: nomic-embed-text
   Endpoint: http://localhost:11434
   Dimensions: 768 (auto-detected)
   Status: Connected

   Model Info:
   - Size: 274 MB
   - Loaded: Yes
   - GPU: Apple Metal
```
