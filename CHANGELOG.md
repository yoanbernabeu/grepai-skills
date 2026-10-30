# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- Skills synced with grepai v1.2.0 behavior changes:
  - `grepai-embeddings-openai`, `grepai-config-reference`, `grepai-init`, `grepai-troubleshooting`, `grepai-workspaces`: environment API keys are only sent to official endpoints; custom endpoints need `embedder.api_key` (`api_key: unused` for keyless local servers)
  - `grepai-config-reference`, `grepai-languages`, `grepai-troubleshooting`: `framework_processing.node_path` restrictions and Vue trust warning
  - `grepai-storage-qdrant`: sharing an explicit `collection` (`project_namespace`), upgrade path, hybrid search fix
  - `grepai-trace-callers`, `grepai-trace-callees`, `grepai-trace-graph`: deterministic same-name resolution, real callees only, no `Class::method`
  - `grepai-watch-daemon`, `grepai-troubleshooting`: one daemon per project/worktree, leftover `*.pid.lock`, log directory per OS
  - `grepai-workspaces`, `grepai-troubleshooting`: RPG link repair in workspaces and with Qdrant

### Fixed

- Removed `${VAR}` substitution examples and `.env` loading claims: grepai reads config values literally and does not load `.env` files
- `grepai-storage-postgres`: replaced `dsn: ${DATABASE_URL}` with a DSN without password plus `PGPASSWORD`
- `grepai-trace-callers`, `grepai-trace-callees`, `grepai-trace-graph`, `grepai-languages`, `grepai-config-reference`, `grepai-init`: removed `--mode precise`, `trace.mode` and `trace.exclude_patterns` (no effect, removed in grepai 1.0.0); default `trace.enabled_languages` updated (Vue, Lua, Ruby, GDScript, Java, F#, `.hxx`)
- `grepai-languages`: rewritten language tables (Ruby, Lua, F#, GDScript and Vue are traced; built-in indexed extensions; `chunking.custom_extensions`)
- `grepai-workspaces`: rewritten for the real `~/.grepai/workspace.yaml` format, PostgreSQL/Qdrant requirement, `workspace create` flags, `grepai index --workspace`, project removal by name

## [1.0.0] - 2026-01-28

### Added

- Initial release with 27 skills organized in 9 categories
- **Getting Started** (3 skills)
  - `grepai-installation` - Multi-platform installation guide
  - `grepai-ollama-setup` - Ollama setup for local embeddings
  - `grepai-quickstart` - 5-minute quickstart guide
- **Configuration** (3 skills)
  - `grepai-init` - Project initialization
  - `grepai-config-reference` - Complete configuration reference
  - `grepai-ignore-patterns` - File exclusion patterns
- **Embeddings** (3 skills)
  - `grepai-embeddings-ollama` - Ollama provider configuration
  - `grepai-embeddings-openai` - OpenAI provider configuration
  - `grepai-embeddings-lmstudio` - LM Studio provider configuration
- **Storage** (3 skills)
  - `grepai-storage-gob` - GOB local file storage
  - `grepai-storage-postgres` - PostgreSQL + pgvector
  - `grepai-storage-qdrant` - Qdrant vector database
- **Indexing** (2 skills)
  - `grepai-watch-daemon` - Watch daemon management
  - `grepai-chunking` - Code chunking configuration
- **Search** (4 skills)
  - `grepai-search-basics` - Basic semantic search
  - `grepai-search-advanced` - Advanced options and JSON output
  - `grepai-search-tips` - Effective query writing
  - `grepai-search-boosting` - Score boosting configuration
- **Trace** (3 skills)
  - `grepai-trace-callers` - Find function callers
  - `grepai-trace-callees` - Find function callees
  - `grepai-trace-graph` - Complete call graphs
- **Integration** (3 skills)
  - `grepai-mcp-claude` - Claude Code MCP integration
  - `grepai-mcp-cursor` - Cursor IDE MCP integration
  - `grepai-mcp-tools` - MCP tools reference
- **Advanced** (3 skills)
  - `grepai-workspaces` - Multi-project workspaces
  - `grepai-languages` - Supported languages
  - `grepai-troubleshooting` - Troubleshooting guide
- 10 skill packs for modular installation
- Support for 20+ AI agents (Claude Code, Cursor, Windsurf, etc.)
