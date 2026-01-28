<p align="center">
  <img src="https://img.shields.io/badge/Skills-27-brightgreen" alt="27 Skills">
  <img src="https://img.shields.io/badge/License-MIT-blue" alt="MIT License">
  <img src="https://img.shields.io/badge/PRs-Welcome-orange" alt="PRs Welcome">
  <img src="https://img.shields.io/github/stars/yoanbernabeu/grepai-skills?style=social" alt="GitHub Stars">
</p>

<h1 align="center">GrepAI Skills</h1>

<p align="center">
  <strong>27 AI Agent Skills to master semantic code search with GrepAI.</strong><br>
  Installation, configuration, search, tracing, MCP integration — everything you need.
</p>

<p align="center">
  <a href="#quick-install">Install</a> •
  <a href="#all-27-skills">Skills</a> •
  <a href="#example-usage">Usage</a> •
  <a href="#contributing">Contribute</a>
</p>

---

## Quick Install

```bash
npx add-skill yoanbernabeu/grepai-skills
```

**That's it.** Works with Claude Code, Cursor, Codex, OpenCode, Windsurf, and 20+ AI agents.

### One-Liners

```bash
# Install everything (27 skills)
npx add-skill yoanbernabeu/grepai-skills

# Install only search skills
npx add-skill yoanbernabeu/grepai-skills --skill grepai-search-basics

# Install globally (available in all projects)
npx add-skill yoanbernabeu/grepai-skills --global

# List all available skills
npx add-skill yoanbernabeu/grepai-skills --list
```

### Other Installation Methods

<details>
<summary><strong>Claude Code Plugin</strong></summary>

```bash
/plugin marketplace add yoanbernabeu/grepai-skills
/plugin install grepai-complete@grepai-skills
```
</details>

<details>
<summary><strong>Manual Installation</strong></summary>

Copy the `skills/` directory to:
- **Global**: `~/.claude/skills/` (or `~/.cursor/skills/`, etc.)
- **Project**: `.claude/skills/` (or `.cursor/skills/`, etc.)
</details>

---

## What is GrepAI?

[GrepAI](https://github.com/yoanbernabeu/grepai) is a semantic code search tool that:

- **Searches by meaning**, not just text patterns
- **100% private** with local embeddings (Ollama)
- **Real-time indexing** with file watching
- **Call graph analysis** to trace function dependencies
- **AI-ready** with native MCP integration

---

## Why These Skills?

Understanding a new codebase is **hard**. Traditional grep finds exact text, but you often don't know the exact words. GrepAI searches by meaning — but it has a learning curve. These skills give your AI agent **expert-level knowledge** of GrepAI so you can find code instantly.

| Challenge | Skill That Helps |
|-----------|------------------|
| "How do I install GrepAI?" | `grepai-installation` |
| "How do I set up Ollama?" | `grepai-ollama-setup` |
| "My searches return poor results" | `grepai-search-tips` |
| "What calls this function?" | `grepai-trace-callers` |
| "How do I use with Claude Code?" | `grepai-mcp-claude` |

---

## All 27 Skills

### Getting Started
| Skill | What It Does |
|-------|--------------|
| `grepai-installation` | Multi-platform installation (Homebrew, shell, Windows) |
| `grepai-ollama-setup` | Install and configure Ollama for local embeddings |
| `grepai-quickstart` | Get searching in 5 minutes |

### Configuration
| Skill | What It Does |
|-------|--------------|
| `grepai-init` | Initialize GrepAI in a project |
| `grepai-config-reference` | Complete configuration reference |
| `grepai-ignore-patterns` | Exclude files and directories from indexing |

### Embeddings Providers
| Skill | What It Does |
|-------|--------------|
| `grepai-embeddings-ollama` | Configure Ollama for local, private embeddings |
| `grepai-embeddings-openai` | Configure OpenAI for cloud embeddings |
| `grepai-embeddings-lmstudio` | Configure LM Studio with GUI interface |

### Storage Backends
| Skill | What It Does |
|-------|--------------|
| `grepai-storage-gob` | Local file storage (default, simple) |
| `grepai-storage-postgres` | PostgreSQL + pgvector for teams |
| `grepai-storage-qdrant` | Qdrant for high-performance search |

### Indexing
| Skill | What It Does |
|-------|--------------|
| `grepai-watch-daemon` | Configure and manage the watch daemon |
| `grepai-chunking` | Optimize how code is split for embedding |

### Semantic Search
| Skill | What It Does |
|-------|--------------|
| `grepai-search-basics` | Basic semantic code search |
| `grepai-search-advanced` | JSON output, compact mode, AI integration |
| `grepai-search-tips` | Write effective search queries |
| `grepai-search-boosting` | Prioritize source code over tests |

### Call Graph Analysis
| Skill | What It Does |
|-------|--------------|
| `grepai-trace-callers` | Find all callers of a function |
| `grepai-trace-callees` | Find all functions called by a function |
| `grepai-trace-graph` | Build complete dependency graphs |

### AI Agent Integration
| Skill | What It Does |
|-------|--------------|
| `grepai-mcp-claude` | Integrate with Claude Code via MCP |
| `grepai-mcp-cursor` | Integrate with Cursor IDE via MCP |
| `grepai-mcp-tools` | Reference for all MCP tools |

### Advanced
| Skill | What It Does |
|-------|--------------|
| `grepai-workspaces` | Multi-project workspace management |
| `grepai-languages` | Supported programming languages |
| `grepai-troubleshooting` | Diagnose and fix common issues |

---

## Skill Packs

Install skills by category:

| Pack | Skills | Description |
|------|--------|-------------|
| `grepai-getting-started` | 3 | Installation, Ollama setup, quickstart |
| `grepai-configuration` | 3 | Init, config reference, ignore patterns |
| `grepai-embeddings` | 3 | Ollama, OpenAI, LM Studio providers |
| `grepai-storage` | 3 | GOB, PostgreSQL, Qdrant backends |
| `grepai-indexing` | 2 | Watch daemon, chunking |
| `grepai-search` | 4 | Basics, advanced, tips, boosting |
| `grepai-trace` | 3 | Callers, callees, graphs |
| `grepai-integration` | 3 | Claude Code, Cursor, MCP tools |
| `grepai-advanced` | 3 | Workspaces, languages, troubleshooting |
| **`grepai-complete`** | **27** | **All skills — complete toolkit** |

---

## Supported AI Agents

| Agent | Project Path | Global Path |
|-------|--------------|-------------|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Codex | `.codex/skills/` | `~/.codex/skills/` |
| OpenCode | `.opencode/skill/` | `~/.config/opencode/skill/` |
| Windsurf | `.windsurf/skills/` | `~/.windsurf/skills/` |

---

## Example Usage

Just ask your AI agent:

```
"Help me install and configure GrepAI"

"Search for authentication code in this project"

"What functions call the Login function?"

"Set up GrepAI with Claude Code via MCP"

"Why are my search results poor?"

"Configure GrepAI to use OpenAI embeddings"

"How do I search across multiple projects?"
```

---

## Quick Start Guide

1. **Install**: `npx add-skill yoanbernabeu/grepai-skills`
2. **Setup GrepAI**: Follow `grepai-installation` skill
3. **Setup Ollama**: Follow `grepai-ollama-setup` skill
4. **Initialize**: `grepai init` in your project
5. **Index**: `grepai watch` to build the index
6. **Search**: `grepai search "your query"`
7. **Integrate**: Add MCP for your AI agent

---

## Quick Reference

### Essential Commands

```bash
# Initialize project
grepai init

# Start indexing
grepai watch

# Search semantically
grepai search "user authentication"

# Find callers
grepai trace callers "Login"

# Check status
grepai status
```

### Search Examples

```bash
# Basic search
grepai search "error handling"

# Limit results
grepai search "database queries" --limit 5

# JSON for AI agents
grepai search "authentication" --json --compact
```

---

## Resources

- [GrepAI Documentation](https://yoanbernabeu.github.io/grepai/)
- [GrepAI GitHub](https://github.com/yoanbernabeu/grepai)
- [add-skill CLI Documentation](https://add-skill.org)

---

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

```bash
# Fork, clone, and create a branch
git checkout -b feature/new-skill

# Add your skill following the SKILL.md format
# Submit a pull request
```

---

## License

[MIT](LICENSE) — Feel free to use, modify, and distribute.

---

<p align="center">
  <strong>Built by the community, for the community.</strong><br><br>
  <a href="https://github.com/yoanbernabeu/grepai-skills">Star this repo</a> if it helps you search code!
</p>
