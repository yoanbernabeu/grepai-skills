---
name: grepai-watch-daemon
description: Configure and manage the GrepAI watch daemon. Use this skill for real-time code indexing and file monitoring.
---

# GrepAI Watch Daemon

This skill covers the `grepai watch` command and daemon management for real-time code indexing.

## When to Use This Skill

- Starting initial code indexing
- Setting up real-time file monitoring
- Running the daemon in background
- Troubleshooting indexing issues

## What the Watch Daemon Does

The watch daemon:
1. **Scans** all source files in your project
2. **Chunks** code into segments (~512 tokens)
3. **Generates** embeddings via your configured provider
4. **Stores** vectors in your configured backend
5. **Monitors** for file changes in real-time
6. **Updates** the index when files change

## Basic Usage

### Start Watching (Foreground)

```bash
cd /your/project
grepai watch
```

Output:
```
🔍 GrepAI Watch
   Scanning files...
   Found 245 files
   Processing chunks...
   ████████████████████████████████ 100%
   Indexed 1,234 chunks
   Watching for changes...
```

Press `Ctrl+C` to stop.

### Start in Background

```bash
grepai watch --background
```

Output:
```
🔍 GrepAI Watch (background)
   Daemon started with PID 12345
   Log file: ~/.grepai/daemon.log
```

### Check Daemon Status

```bash
grepai watch --status
```

Output:
```
✅ GrepAI Daemon Running

   PID: 12345
   Started: 2025-01-28 10:30:00
   Project: /path/to/project

   Statistics:
   - Files indexed: 245
   - Chunks: 1,234
   - Last update: 2 minutes ago
```

### Stop the Daemon

```bash
grepai watch --stop
```

Output:
```
✅ Daemon stopped (PID 12345)
```

## Command Reference

| Command | Description |
|---------|-------------|
| `grepai watch` | Start daemon in foreground |
| `grepai watch --background` | Start daemon in background |
| `grepai watch --status` | Check daemon status |
| `grepai watch --stop` | Stop running daemon |

## Configuration

### Watch Settings

```yaml
# .grepai/config.yaml
watch:
  # Debounce delay in milliseconds
  # Groups rapid file changes together
  debounce_ms: 500
```

### Debounce Explained

When you save a file, editors often write multiple times quickly. Debouncing waits for changes to settle:

| Value | Behavior |
|-------|----------|
| `100` | More responsive, more reindexing |
| `500` | Balanced (default) |
| `1000` | Less responsive, fewer reindexing |

## Initial Indexing

### What Gets Indexed

The daemon indexes files:
- Matching supported extensions (.go, .js, .ts, .py, etc.)
- Not in ignore patterns (node_modules, .git, etc.)
- Respecting `.gitignore`

### Indexing Progress

Large codebases show progress:

```
Scanning files...
Found 10,245 files
Processing chunks...
████████████████░░░░░░░░░░░░░░░░ 50% (5,122/10,245)
```

### Indexing Time Estimates

| Codebase | Files | Time (Ollama) | Time (OpenAI) |
|----------|-------|---------------|---------------|
| Small | 100 | ~30s | ~10s |
| Medium | 1,000 | ~5min | ~1min |
| Large | 10,000 | ~30min | ~5min |

## Real-Time Monitoring

After initial indexing, the daemon watches for:
- File creation
- File modification
- File deletion
- File renames

### File Change Detection

Uses OS-native file watching:
- **macOS:** FSEvents
- **Linux:** inotify
- **Windows:** ReadDirectoryChangesW

### What Triggers Reindexing

| Action | Result |
|--------|--------|
| Save existing file | Re-embed file chunks |
| Create new file | Index new chunks |
| Delete file | Remove from index |
| Rename file | Update path, keep vectors |

## Background Daemon Management

### Starting on System Boot

#### macOS (launchd)

Create `~/Library/LaunchAgents/com.grepai.watch.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.grepai.watch</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/grepai</string>
        <string>watch</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/path/to/your/project</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

Load:
```bash
launchctl load ~/Library/LaunchAgents/com.grepai.watch.plist
```

#### Linux (systemd)

Create `~/.config/systemd/user/grepai-watch.service`:

```ini
[Unit]
Description=GrepAI Watch Daemon
After=network.target

[Service]
Type=simple
WorkingDirectory=/path/to/your/project
ExecStart=/usr/local/bin/grepai watch
Restart=always

[Install]
WantedBy=default.target
```

Enable:
```bash
systemctl --user enable grepai-watch
systemctl --user start grepai-watch
```

### Checking Logs

```bash
# Background daemon logs
tail -f ~/.grepai/daemon.log

# Or with systemd
journalctl --user -u grepai-watch -f
```

## Multiple Projects

### One Daemon Per Project

Run separate daemons for each project:

```bash
# Terminal 1: Project A
cd /path/to/project-a
grepai watch --background

# Terminal 2: Project B
cd /path/to/project-b
grepai watch --background
```

### Using Workspaces

For multi-project setups:

```bash
grepai workspace create my-workspace
grepai workspace add my-workspace /path/to/project-a
grepai workspace add my-workspace /path/to/project-b
grepai watch --workspace my-workspace
```

## Troubleshooting

### Daemon Won't Start

❌ **Problem:** "Another daemon is already running"
✅ **Solution:**
```bash
grepai watch --stop
grepai watch --background
```

❌ **Problem:** "Config not found"
✅ **Solution:** Initialize first:
```bash
grepai init
grepai watch
```

❌ **Problem:** "Embedder connection failed"
✅ **Solution:** Start your embedding provider:
```bash
ollama serve  # For Ollama
```

### Indexing Issues

❌ **Problem:** Files not being indexed
✅ **Solution:** Check ignore patterns in config, ensure file extension is supported

❌ **Problem:** Indexing very slow
✅ **Solutions:**
- Use OpenAI for faster cloud embeddings
- Add more ignore patterns
- Increase chunking size

❌ **Problem:** Index seems outdated
✅ **Solution:** Clear and reindex:
```bash
rm .grepai/index.gob
grepai watch
```

### File Watch Issues

❌ **Problem:** Changes not detected
✅ **Solutions:**
- Reduce debounce_ms
- Check inotify limits (Linux):
  ```bash
  echo 65536 | sudo tee /proc/sys/fs/inotify/max_user_watches
  ```

## Best Practices

1. **Run in background:** For continuous monitoring
2. **Use workspace for monorepos:** Better organization
3. **Set up auto-start:** launchd or systemd
4. **Check logs periodically:** Monitor for errors
5. **Reindex after config changes:** Especially after changing embedding model

## Status Check

Regular health check:

```bash
grepai status
```

Output:
```
✅ GrepAI Status

   Project: /path/to/project
   Config: .grepai/config.yaml

   Embedder: Ollama (nomic-embed-text)
   Storage: GOB (.grepai/index.gob)

   Index:
   - Files: 245
   - Chunks: 1,234
   - Size: 12.5 MB
   - Last updated: 2025-01-28 10:30:00

   Daemon: Running (PID 12345)
```

## Output Format

Watch daemon status:

```
✅ Watch Daemon Active

   Mode: Background
   PID: 12345
   Project: /path/to/project

   Initial Index:
   - Files scanned: 245
   - Chunks created: 1,234
   - Duration: 45s

   Real-time Monitor:
   - Debounce: 500ms
   - Events processed: 23
   - Last event: 5 minutes ago

   Next steps:
   - Run 'grepai search "query"' to search
   - Run 'grepai watch --status' to check status
   - Run 'grepai watch --stop' to stop daemon
```
