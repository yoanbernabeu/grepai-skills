---
name: grepai-storage-postgres
description: Configure PostgreSQL with pgvector for GrepAI. Use this skill for team environments and large codebases.
---

# GrepAI Storage with PostgreSQL

This skill covers using PostgreSQL with the pgvector extension as the storage backend for GrepAI.

## When to Use This Skill

- Team environments with shared index
- Large codebases (10K+ files)
- Need concurrent access
- Integration with existing PostgreSQL infrastructure

## Prerequisites

1. PostgreSQL 14+ with pgvector extension
2. Database user with create table permissions
3. Network access to PostgreSQL server

## Advantages

| Benefit | Description |
|---------|-------------|
| 👥 **Team sharing** | Multiple users can access same index |
| 📏 **Scalable** | Handles large codebases |
| 🔄 **Concurrent** | Multiple simultaneous searches |
| 💾 **Persistent** | Data survives machine restarts |
| 🔧 **Familiar** | Standard database tooling |

## Setting Up PostgreSQL with pgvector

### Option 1: Docker (Recommended for Development)

```bash
# Run PostgreSQL with pgvector
docker run -d \
  --name grepai-postgres \
  -e POSTGRES_USER=grepai \
  -e POSTGRES_PASSWORD=grepai \
  -e POSTGRES_DB=grepai \
  -p 5432:5432 \
  pgvector/pgvector:pg16
```

### Option 2: Install on Existing PostgreSQL

```bash
# Install pgvector extension (Ubuntu/Debian)
sudo apt install postgresql-16-pgvector

# Or compile from source
git clone https://github.com/pgvector/pgvector.git
cd pgvector
make
sudo make install
```

Then enable the extension:

```sql
-- Connect to your database
CREATE EXTENSION IF NOT EXISTS vector;
```

### Option 3: Managed Services

- **Supabase:** pgvector included by default
- **Neon:** pgvector available
- **AWS RDS:** Install pgvector extension
- **Azure Database:** pgvector available

## Configuration

### Basic Configuration

```yaml
# .grepai/config.yaml
store:
  backend: postgres
  postgres:
    dsn: postgres://user:password@localhost:5432/grepai
```

### With Environment Variable

```yaml
store:
  backend: postgres
  postgres:
    dsn: ${DATABASE_URL}
```

Set the environment variable:
```bash
export DATABASE_URL="postgres://user:password@localhost:5432/grepai"
```

### Full DSN Options

```yaml
store:
  backend: postgres
  postgres:
    dsn: postgres://user:password@host:5432/database?sslmode=require
```

DSN components:
- `user`: Database username
- `password`: Database password
- `host`: Server hostname or IP
- `5432`: Port (default: 5432)
- `database`: Database name
- `sslmode`: SSL mode (disable, require, verify-full)

## SSL Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `disable` | No SSL | Local development |
| `require` | SSL required | Production |
| `verify-full` | SSL + verify certificate | High security |

```yaml
# Production with SSL
store:
  backend: postgres
  postgres:
    dsn: postgres://user:pass@prod.db.com:5432/grepai?sslmode=require
```

## Database Schema

GrepAI automatically creates these tables:

```sql
-- Vector embeddings table
CREATE TABLE IF NOT EXISTS embeddings (
    id SERIAL PRIMARY KEY,
    file_path TEXT NOT NULL,
    chunk_index INTEGER NOT NULL,
    content TEXT NOT NULL,
    start_line INTEGER,
    end_line INTEGER,
    embedding vector(768),  -- Dimension matches your model
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(file_path, chunk_index)
);

-- Index for vector similarity search
CREATE INDEX ON embeddings USING ivfflat (embedding vector_cosine_ops);
```

## Verifying Setup

### Check pgvector Extension

```sql
-- Connect to database
psql -U grepai -d grepai

-- Check extension is installed
SELECT * FROM pg_extension WHERE extname = 'vector';

-- Check GrepAI tables exist (after first grepai watch)
\dt
```

### Test Connection from GrepAI

```bash
# Check status
grepai status

# Should show PostgreSQL backend info
```

## Performance Tuning

### PostgreSQL Configuration

For better vector search performance:

```sql
-- Increase work memory for vector operations
SET work_mem = '256MB';

-- Adjust for your hardware
SET effective_cache_size = '4GB';
SET shared_buffers = '1GB';
```

### Index Tuning

For large indices, tune the IVFFlat index:

```sql
-- More lists = faster search, more memory
CREATE INDEX ON embeddings
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);  -- Adjust based on row count
```

Rule of thumb: `lists = sqrt(rows)`

## Concurrent Access

PostgreSQL handles concurrent access automatically:

- Multiple `grepai search` commands work simultaneously
- One `grepai watch` daemon per codebase
- Many users can share the same index

## Team Setup

### Shared Database

All team members point to the same database:

```yaml
# Each developer's .grepai/config.yaml
store:
  backend: postgres
  postgres:
    dsn: postgres://team:secret@shared-db.company.com:5432/grepai
```

### Per-Project Databases

For isolated projects, use separate databases:

```bash
# Create databases
createdb -U postgres grepai_projecta
createdb -U postgres grepai_projectb
```

```yaml
# Project A config
store:
  backend: postgres
  postgres:
    dsn: postgres://user:pass@localhost:5432/grepai_projecta
```

## Backup and Restore

### Backup

```bash
pg_dump -U grepai -d grepai > grepai_backup.sql
```

### Restore

```bash
psql -U grepai -d grepai < grepai_backup.sql
```

## Migrating from GOB

1. Set up PostgreSQL with pgvector
2. Update configuration:
```yaml
store:
  backend: postgres
  postgres:
    dsn: postgres://user:pass@localhost:5432/grepai
```
3. Delete old index:
```bash
rm .grepai/index.gob
```
4. Re-index:
```bash
grepai watch
```

## Common Issues

❌ **Problem:** `FATAL: password authentication failed`
✅ **Solution:** Check DSN credentials and pg_hba.conf

❌ **Problem:** `ERROR: extension "vector" is not available`
✅ **Solution:** Install pgvector:
```bash
sudo apt install postgresql-16-pgvector
# Then: CREATE EXTENSION vector;
```

❌ **Problem:** `ERROR: type "vector" does not exist`
✅ **Solution:** Enable extension in the database:
```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

❌ **Problem:** Connection refused
✅ **Solution:**
- Check PostgreSQL is running
- Verify host and port
- Check firewall rules

❌ **Problem:** Slow searches
✅ **Solution:**
- Add IVFFlat index
- Increase `work_mem`
- Vacuum and analyze tables

## Best Practices

1. **Use environment variables:** Don't commit credentials
2. **Enable SSL:** For remote databases
3. **Regular backups:** pg_dump before major changes
4. **Monitor performance:** Check query times
5. **Index maintenance:** Regular VACUUM ANALYZE

## Output Format

PostgreSQL storage status:

```
✅ PostgreSQL Storage Configured

   Backend: PostgreSQL + pgvector
   Host: localhost:5432
   Database: grepai
   SSL: disabled

   Contents:
   - Files: 2,450
   - Chunks: 12,340
   - Vector dimension: 768

   Performance:
   - Connection: OK
   - IVFFlat index: Yes
   - Search latency: ~50ms
```
