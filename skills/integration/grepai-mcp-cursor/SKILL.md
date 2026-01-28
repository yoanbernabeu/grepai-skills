---
name: grepai-mcp-cursor
description: Integrate GrepAI with Cursor IDE via MCP. Use this skill to enable semantic code search in Cursor.
---

# GrepAI MCP Integration with Cursor

This skill covers integrating GrepAI with Cursor IDE using the Model Context Protocol (MCP).

## When to Use This Skill

- Setting up GrepAI in Cursor
- Enabling semantic search for Cursor AI
- Configuring MCP for Cursor
- Troubleshooting Cursor integration

## What is Cursor?

Cursor is an AI-powered IDE that supports MCP for external tools. GrepAI integration gives Cursor's AI:

- Semantic code search beyond simple text matching
- Call graph analysis for understanding dependencies
- Index-based code navigation

## Prerequisites

1. GrepAI installed
2. Ollama running (or other embedding provider)
3. Project indexed (`grepai watch`)
4. Cursor IDE installed

## Configuration

### Step 1: Create MCP Config File

Create `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve"]
    }
  }
}
```

### Step 2: Restart Cursor

Close and reopen Cursor for the config to take effect.

### Step 3: Verify

Ask Cursor's AI:
> "Search the codebase for authentication"

Cursor should use the `grepai_search` tool.

## Global Configuration

For GrepAI in all Cursor projects, use global config:

### Location

- **macOS:** `~/.cursor/mcp.json`
- **Linux:** `~/.cursor/mcp.json`
- **Windows:** `%APPDATA%\Cursor\mcp.json`

### Content

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve"]
    }
  }
}
```

## Per-Project Configuration

For project-specific settings:

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve"],
      "cwd": "/absolute/path/to/project"
    }
  }
}
```

## Available Tools

Once configured, Cursor has access to:

| Tool | Description |
|------|-------------|
| `grepai_search` | Semantic code search |
| `grepai_trace_callers` | Find function callers |
| `grepai_trace_callees` | Find function callees |
| `grepai_trace_graph` | Build call graphs |
| `grepai_index_status` | Check index health |

## Usage Examples

### Finding Code

Ask Cursor:
> "Find code that handles user login"

Cursor uses `grepai_search` to find semantically related code.

### Understanding Dependencies

Ask Cursor:
> "What functions call validateToken?"

Cursor uses `grepai_trace_callers` to show all callers.

### Code Navigation

Ask Cursor:
> "Show me the call graph for processPayment"

Cursor uses `grepai_trace_graph` to display dependencies.

## Cursor Settings Integration

### Enable MCP in Settings

1. Open Cursor Settings (`Cmd+,` / `Ctrl+,`)
2. Search for "MCP"
3. Ensure MCP is enabled

### Verify MCP Status

1. Open Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)
2. Search "MCP"
3. Check connected servers

## Windsurf Configuration

Windsurf uses the same MCP format as Cursor:

### Location

Create `.windsurf/mcp.json`:

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve"]
    }
  }
}
```

## Multiple Projects Setup

### Option 1: Separate Configs

Each project has its own `.cursor/mcp.json` with appropriate `cwd`.

### Option 2: Workspaces

```bash
# Create workspace
grepai workspace create dev
grepai workspace add dev /path/to/project1
grepai workspace add dev /path/to/project2
```

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve", "--workspace", "dev"]
    }
  }
}
```

## Environment Variables

If GrepAI uses environment variables (like API keys):

```json
{
  "mcpServers": {
    "grepai": {
      "command": "grepai",
      "args": ["mcp-serve"],
      "env": {
        "OPENAI_API_KEY": "sk-..."
      }
    }
  }
}
```

**Better:** Set environment variables in your shell profile instead.

## Troubleshooting

### MCP Not Recognized

❌ **Problem:** Cursor doesn't see GrepAI tools

✅ **Solutions:**
1. Check file location: `.cursor/mcp.json` in project root
2. Verify JSON syntax (no trailing commas)
3. Restart Cursor completely
4. Check `grepai` is in PATH

### Search Returns Nothing

❌ **Problem:** Empty search results

✅ **Solutions:**
1. Ensure index exists: `grepai status`
2. Run `grepai watch` first
3. Verify working directory

### Connection Errors

❌ **Problem:** MCP connection failed

✅ **Solutions:**
1. Test manually: `grepai mcp-serve`
2. Check Ollama: `curl http://localhost:11434/api/tags`
3. Look at Cursor's developer console for errors

### Wrong Results

❌ **Problem:** Results from wrong project

✅ **Solutions:**
1. Set explicit `cwd` in config
2. Check you opened the right folder in Cursor
3. Use `grepai_index_status` to verify

## Performance Tips

1. **Background daemon:** Keep `grepai watch --background` running
2. **Use compact mode:** MCP tools use compact by default
3. **Limit results:** AI will request appropriate limits
4. **Index regularly:** Especially after git pull

## Comparison: Cursor vs Claude Code

| Feature | Cursor | Claude Code |
|---------|--------|-------------|
| Config location | `.cursor/mcp.json` | `~/.claude/mcp.json` |
| Setup command | Manual JSON | `claude mcp add` |
| Project scope | Per-project or global | Global |
| IDE integration | Native | Terminal |

## Best Practices

1. **Version control:** Add `.cursor/mcp.json` to git (without secrets)
2. **Team setup:** Document MCP config in README
3. **Keep index fresh:** Run watch daemon
4. **Test locally:** Verify `grepai mcp-serve` works first
5. **Use workspaces:** For multi-project setups

## Removing Integration

Delete `.cursor/mcp.json` and restart Cursor.

Or remove just GrepAI:
```json
{
  "mcpServers": {
    // Remove grepai entry
  }
}
```

## Output Format

Successful Cursor setup:

```
✅ GrepAI MCP Integration for Cursor

   Config: .cursor/mcp.json
   Server: grepai mcp-serve
   Status: Ready

   Available tools:
   - grepai_search
   - grepai_trace_callers
   - grepai_trace_callees
   - grepai_trace_graph
   - grepai_index_status

   Cursor AI can now search your code semantically!

   Test: Ask Cursor "search for authentication code"
```
