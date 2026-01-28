---
name: grepai-storage-qdrant
description: Configure Qdrant vector database for GrepAI. Use this skill for high-performance vector search.
---

# GrepAI Storage with Qdrant

This skill covers using Qdrant as the storage backend for GrepAI, offering high-performance vector search.

## When to Use This Skill

- Need fastest possible search performance
- Very large codebases (50K+ files)
- Already using Qdrant infrastructure
- Want advanced vector search features

## What is Qdrant?

Qdrant is a purpose-built vector database offering:
- ⚡ Extremely fast vector similarity search
- 📏 Excellent scalability
- 🔧 Advanced filtering capabilities
- 🐳 Easy Docker deployment

## Prerequisites

1. Qdrant server running
2. Network access to Qdrant

## Advantages

| Benefit | Description |
|---------|-------------|
| ⚡ **Performance** | Fastest vector search |
| 📏 **Scalability** | Handles millions of vectors |
| 🔍 **Advanced** | Filtering, payloads, sharding |
| 🐳 **Easy deploy** | Docker-ready |
| ☁️ **Cloud option** | Qdrant Cloud available |

## Setting Up Qdrant

### Option 1: Docker (Recommended)

```bash
# Run Qdrant with persistent storage
docker run -d \
  --name grepai-qdrant \
  -p 6333:6333 \
  -p 6334:6334 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

Ports:
- `6333`: REST API
- `6334`: gRPC API (used by GrepAI)

### Option 2: Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_storage:/qdrant/storage
    environment:
      - QDRANT__SERVICE__GRPC_PORT=6334

volumes:
  qdrant_storage:
```

```bash
docker-compose up -d
```

### Option 3: Qdrant Cloud

1. Sign up at [cloud.qdrant.io](https://cloud.qdrant.io)
2. Create a cluster
3. Get your endpoint and API key

## Configuration

### Basic Configuration (Local)

```yaml
# .grepai/config.yaml
store:
  backend: qdrant
  qdrant:
    endpoint: localhost
    port: 6334
```

### With TLS (Production)

```yaml
store:
  backend: qdrant
  qdrant:
    endpoint: qdrant.company.com
    port: 6334
    use_tls: true
```

### With API Key (Qdrant Cloud)

```yaml
store:
  backend: qdrant
  qdrant:
    endpoint: your-cluster.aws.cloud.qdrant.io
    port: 6334
    use_tls: true
    api_key: ${QDRANT_API_KEY}
```

Set the environment variable:
```bash
export QDRANT_API_KEY="your-api-key"
```

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `endpoint` | `localhost` | Qdrant server hostname |
| `port` | `6334` | gRPC port |
| `use_tls` | `false` | Enable TLS encryption |
| `api_key` | none | Authentication key |

## Verifying Setup

### Check Qdrant is Running

```bash
# REST API health check
curl http://localhost:6333/health

# Expected: {"status":"ok"}
```

### Check Collections (after indexing)

```bash
# List collections
curl http://localhost:6333/collections

# Get collection info
curl http://localhost:6333/collections/grepai
```

### From GrepAI

```bash
grepai status

# Should show Qdrant backend info
```

## Qdrant Dashboard

Access the web dashboard at `http://localhost:6333/dashboard`:
- View collections
- Browse vectors
- Execute queries
- Monitor performance

## Performance Characteristics

### Search Latency

| Codebase Size | Vectors | Search Time |
|---------------|---------|-------------|
| Small (1K files) | 5,000 | <10ms |
| Medium (10K files) | 50,000 | <20ms |
| Large (100K files) | 500,000 | <50ms |

### Memory Usage

Qdrant loads vectors into memory for fast search:

| Vectors | Dimensions | Memory |
|---------|------------|--------|
| 10,000 | 768 | ~60 MB |
| 100,000 | 768 | ~600 MB |
| 1,000,000 | 768 | ~6 GB |

## Advanced Configuration

### Qdrant Server Configuration

Create `config/production.yaml`:

```yaml
storage:
  storage_path: /qdrant/storage

service:
  grpc_port: 6334
  http_port: 6333
  max_request_size_mb: 32

optimizers:
  memmap_threshold_kb: 200000
  indexing_threshold_kb: 50000
```

Mount in Docker:
```bash
docker run -d \
  -v ./config:/qdrant/config \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

### Collection Settings

GrepAI creates a collection named `grepai` with:
- Vector size: matches your embedding dimensions
- Distance: Cosine similarity
- On-disk storage for large datasets

## Clustering (Advanced)

For very large deployments, Qdrant supports distributed mode:

```yaml
# qdrant config
cluster:
  enabled: true
  p2p:
    port: 6335
```

## Backup and Restore

### Snapshot Creation

```bash
# Create snapshot via REST API
curl -X POST 'http://localhost:6333/collections/grepai/snapshots'
```

### Restore Snapshot

```bash
# Restore from snapshot
curl -X PUT 'http://localhost:6333/collections/grepai/snapshots/recover' \
  -H 'Content-Type: application/json' \
  -d '{"location": "/path/to/snapshot"}'
```

## Migrating from GOB

1. Start Qdrant:
```bash
docker run -d --name qdrant -p 6333:6333 -p 6334:6334 qdrant/qdrant
```

2. Update configuration:
```yaml
store:
  backend: qdrant
  qdrant:
    endpoint: localhost
    port: 6334
```

3. Delete old index:
```bash
rm .grepai/index.gob
```

4. Re-index:
```bash
grepai watch
```

## Migrating from PostgreSQL

1. Start Qdrant
2. Update configuration to use Qdrant
3. Re-index (embeddings must be regenerated)

## Common Issues

❌ **Problem:** Connection refused
✅ **Solution:** Ensure Qdrant is running:
```bash
docker ps | grep qdrant
docker start grepai-qdrant
```

❌ **Problem:** gRPC connection failed
✅ **Solution:** Check port 6334 is exposed:
```bash
docker run -p 6334:6334 ...
```

❌ **Problem:** Authentication failed
✅ **Solution:** Check API key:
```bash
echo $QDRANT_API_KEY
```

❌ **Problem:** Out of memory
✅ **Solutions:**
- Enable on-disk storage in Qdrant config
- Increase Docker memory limit
- Use Qdrant Cloud for managed scaling

❌ **Problem:** Slow initial indexing
✅ **Solution:** This is normal; Qdrant optimizes in background. Searches will be fast after indexing completes.

## Qdrant vs PostgreSQL

| Feature | Qdrant | PostgreSQL |
|---------|--------|------------|
| Search speed | ⚡⚡⚡ | ⚡⚡ |
| Setup complexity | Easy (Docker) | Medium |
| SQL queries | ❌ | ✅ |
| Scalability | Excellent | Good |
| Memory efficiency | Excellent | Good |
| Team familiarity | Lower | Higher |

**Recommendation:** Use Qdrant for large codebases or maximum performance. Use PostgreSQL if you need SQL integration or team is familiar with it.

## Best Practices

1. **Use persistent volume:** Mount `/qdrant/storage`
2. **Enable TLS in production:** Set `use_tls: true`
3. **Secure API key:** Use environment variables
4. **Monitor memory:** Vector search is memory-intensive
5. **Regular snapshots:** Backup before major changes

## Output Format

Qdrant storage status:

```
✅ Qdrant Storage Configured

   Backend: Qdrant
   Endpoint: localhost:6334
   TLS: disabled
   Collection: grepai

   Contents:
   - Files: 5,000
   - Vectors: 25,000
   - Dimensions: 768

   Performance:
   - Connection: OK
   - Indexed: Yes
   - Search latency: ~15ms
```
