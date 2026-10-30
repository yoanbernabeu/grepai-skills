---
name: grepai-workspaces
description: Configure multi-project workspaces in GrepAI. Use this skill for monorepos and multiple related projects.
---

# GrepAI Workspaces

This skill covers using GrepAI workspaces to manage multiple related projects with a unified search index.

## When to Use This Skill

- Working with monorepos
- Searching across multiple related projects
- Managing microservices architecture
- Organizing large codebases

## What are Workspaces?

Workspaces allow you to:
- Group multiple projects together
- Index them into one shared vector store
- Search or trace across all projects at once
- Or search specific projects within the workspace

## Prerequisites

Workspaces need a vector store that several projects can share:

- **PostgreSQL** with pgvector
- **Qdrant**

The GOB backend is not supported: it is a local file per project, so it cannot be shared.

## Creating a Workspace

```bash
# Interactive prompts (backend, then embedding provider)
grepai workspace create my-workspace

# Qdrant + Ollama, all defaults
grepai workspace create my-workspace --yes

# Qdrant + OpenAI (API key read from OPENAI_API_KEY, official endpoint)
grepai workspace create my-workspace --backend qdrant --provider openai --model text-embedding-3-small

# PostgreSQL + Ollama
grepai workspace create my-workspace --backend postgres --dsn "postgres://grepai:grepai@localhost:5432/grepai"

# From a YAML or JSON file (store + embedder)
grepai workspace create my-workspace --from workspace-config.yaml
```

| Flag | Description | Default |
|------|-------------|---------|
| `--backend` | `qdrant` or `postgres` | Required, unless `--yes` (then `qdrant`) |
| `--provider` | `ollama`, `openai` or `lmstudio` | `ollama` |
| `--model` | Embedding model | `text-embedding-3-small` for `openai`, `nomic-embed-text` otherwise |
| `--endpoint` | Embedder endpoint | Provider default |
| `--dsn` | PostgreSQL connection string | `postgres://localhost:5432/grepai` |
| `--qdrant-endpoint` | Qdrant host | `http://localhost` |
| `--qdrant-port` | Qdrant gRPC port | `6334` |
| `--collection` | Qdrant collection name | `workspace_<name>` |
| `--yes` | Use defaults for every value not set | |
| `--from` | Read `store` and `embedder` from a file (other flags are ignored) | |
| `--ui` | Interactive full-screen wizard | |

## Adding Projects

```bash
grepai workspace add my-workspace /path/to/frontend
grepai workspace add my-workspace /path/to/backend
grepai workspace add my-workspace ./shared-lib
```

Relative paths are converted to absolute paths. The project name is the directory name (`frontend`, `backend`) and must be unique in the workspace. A project does not need its own `grepai init`: without a `.grepai/config.yaml`, it uses default settings.

## Managing Workspaces

```bash
grepai workspace list                     # list workspaces
grepai workspace show my-workspace        # projects and settings
grepai workspace status my-workspace      # checks that each project path exists
grepai workspace remove my-workspace frontend   # remove a project, by name (not path)
grepai workspace delete my-workspace      # remove the workspace entry
```

`workspace delete` only removes the entry from `workspace.yaml`: indexed data stays in PostgreSQL or Qdrant, and project files are untouched.

## Indexing a Workspace

```bash
# Index, then keep watching (foreground)
grepai watch --workspace my-workspace

# Same, in the background
grepai watch --workspace my-workspace --background
grepai watch --workspace my-workspace --status
grepai watch --workspace my-workspace --stop

# Index once and exit (CI, git hooks)
grepai index --workspace my-workspace
```

Chunks go to the shared store. Each project's symbol index (and RPG graph, if enabled) is written to `<project>/.grepai/`. `grepai index --workspace` continues past a failing project and returns an aggregate error. A project folder that is missing is skipped with a warning.

## Searching Workspaces

### Search All Projects

```bash
grepai search --workspace my-workspace "user authentication"
```

### Search Specific Projects

```bash
# One project
grepai search --workspace my-workspace --project frontend "form validation"

# Several projects (repeat --project)
grepai search --workspace my-workspace --project frontend --project backend "query"

# Path prefix inside one project
grepai search --workspace my-workspace --project frontend --path components/ "React hooks"

# Path prefix without --project: start with the project name
grepai search --workspace my-workspace --path frontend/src/ "query"
```

With exactly one `--project`, a relative `--path` is relative to that project; otherwise it must start with a project name.

Workspace search uses the default search boost and does not use hybrid search or RPG enrichment.

## Workspace Configuration

All workspaces are stored in a single file, `~/.grepai/workspace.yaml` (file mode `0600`, since it may hold API keys):

```yaml
version: 1
workspaces:
  my-workspace:
    name: my-workspace
    store:
      backend: qdrant
      qdrant:
        endpoint: localhost
        port: 6334
    embedder:
      provider: ollama
      model: nomic-embed-text
      endpoint: http://localhost:11434
      dimensions: 768
    chunking:
      custom_extensions: [".tpl"]   # optional, merged with each project's list
    projects:
      - name: frontend
        path: /path/to/frontend
      - name: backend
        path: /path/to/backend
```

### API Keys With a Custom Endpoint

Environment keys (`OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `SYNTHETIC_API_KEY`, `REQUESTY_API_KEY`) are only sent to the provider's official endpoint. A workspace embedder with a custom `endpoint` (Azure, LiteLLM, vLLM, a proxy…) needs `api_key` in `~/.grepai/workspace.yaml`; use `api_key: unused` for a local server without authentication:

```yaml
workspaces:
  my-workspace:
    embedder:
      provider: openai
      model: text-embedding-3-small
      endpoint: https://your-resource.openai.azure.com/openai/v1
      api_key: your-key
```

### RPG in Workspaces

Before grepai v1.2.0, RPG symbol↔chunk links were empty in workspace mode, and with the Qdrant backend (workspace or single project) all chunks formed a single RPG node. After upgrading, the next `grepai index --workspace <name>` or `grepai watch --workspace <name>` repairs them, without re-embedding; no manual cleanup is needed.

## MCP with Workspaces

Start the MCP server with `--workspace` so every tool with a `workspace` parameter defaults to it. No local `.grepai/` is needed in this mode:

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve", "--workspace", "my-workspace"]
    }
  }
}
```

Without `--workspace`, agents pass `workspace` (and optionally `projects`) explicitly; `grepai_list_workspaces` and `grepai_list_projects` help them discover valid names.

## Use Cases

### Monorepo

```bash
grepai workspace create monorepo --yes
grepai workspace add monorepo /path/to/monorepo/apps/web
grepai workspace add monorepo /path/to/monorepo/apps/mobile
grepai workspace add monorepo /path/to/monorepo/packages/ui
grepai workspace add monorepo /path/to/monorepo/packages/core
```

### Microservices

```bash
grepai workspace create services --yes
grepai workspace add services /path/to/user-service
grepai workspace add services /path/to/order-service
grepai workspace add services /path/to/payment-service
grepai workspace add services /path/to/notification-service
```

### Frontend + Backend

```bash
grepai workspace create fullstack --yes
grepai workspace add fullstack /path/to/frontend
grepai workspace add fullstack /path/to/backend
grepai workspace add fullstack /path/to/shared-types
```

## Cross-Project Search Examples

### Find All API Endpoints

```bash
grepai search --workspace services "REST API endpoint handler"
```

### Find Shared Type Usage

```bash
grepai search --workspace fullstack "User interface definition"
```

### Find Cross-Project Dependencies

```bash
# What calls the shared auth library?
grepai trace callers --workspace fullstack "validateToken"
```

## Best Practices

1. **Name descriptively:** `ecommerce-stack` not `ws1`
2. **Group related projects:** Only projects that you'd search together
3. **One daemon per workspace:** Run `grepai watch --workspace`
4. **Use project filter:** When you know which project to search
5. **Index in CI:** `grepai index --workspace` fails loudly instead of leaving a partial index

## Common Issues

❌ **Problem:** Workspace not found
✅ **Solution:** Check workspace exists: `grepai workspace list`

❌ **Problem:** Workspace refuses the GOB backend
✅ **Solution:** Use PostgreSQL or Qdrant (`--backend qdrant`)

❌ **Problem:** Project paths changed
✅ **Solution:** Remove the project by name, then add the new path:
```bash
grepai workspace remove my-workspace frontend
grepai workspace add my-workspace /new/path/frontend
```

❌ **Problem:** Search returns mixed results
✅ **Solution:** Use `--project` flag to filter:
```bash
grepai search --workspace ws --project backend "query"
```

❌ **Problem:** `API keys from the environment ... are only sent to the official endpoint`
✅ **Solution:** Set `embedder.api_key` for this workspace in `~/.grepai/workspace.yaml` (see above)

## Output Format

Workspace overview:

```
📁 Workspace: my-workspace

Configuration:
- File: ~/.grepai/workspace.yaml
- Store: qdrant (collection workspace_my-workspace)
- Embedder: ollama (nomic-embed-text)

Projects (3):
- frontend    /path/to/frontend
- backend     /path/to/backend
- shared-lib  /path/to/shared-lib

Commands:
- Index: grepai watch --workspace my-workspace
- Search: grepai search --workspace my-workspace "query"
- Status: grepai workspace status my-workspace
```
