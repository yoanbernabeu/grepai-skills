---
name: grepai-installation
description: Multi-platform installation guide for GrepAI. Use this skill when installing GrepAI on macOS, Linux, or Windows.
---

# GrepAI Installation

This skill covers all methods to install GrepAI on any platform.

## When to Use This Skill

- Installing GrepAI for the first time
- Upgrading an existing GrepAI installation
- Building GrepAI from source
- Verifying a successful installation

## Prerequisites

- **macOS/Linux:** Terminal access
- **Windows:** PowerShell
- **From source:** Go 1.24+ installed

## Installation Methods

### Method 1: Homebrew (macOS - Recommended)

The easiest way to install on macOS:

```bash
# Add the tap and install
brew install yoanbernabeu/tap/grepai

# Verify installation
grepai version
```

**Advantages:**
- Automatic updates with `brew upgrade grepai`
- Clean uninstall with `brew uninstall grepai`
- Manages dependencies automatically

### Method 2: Shell Script (Linux/macOS)

Universal installation script:

```bash
# Download and run the installer
curl -sSL https://raw.githubusercontent.com/yoanbernabeu/grepai/main/install.sh | sh

# Verify installation
grepai version
```

**What the script does:**
1. Detects your OS and architecture
2. Downloads the appropriate binary
3. Installs to `/usr/local/bin/`
4. Sets executable permissions

### Method 3: PowerShell (Windows)

Native Windows installation:

```powershell
# Run in PowerShell (Admin recommended)
irm https://raw.githubusercontent.com/yoanbernabeu/grepai/main/install.ps1 | iex

# Verify installation
grepai version
```

**Note:** You may need to adjust execution policy:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Method 4: Build from Source

For developers or custom builds:

```bash
# Clone the repository
git clone https://github.com/yoanbernabeu/grepai.git
cd grepai

# Build the binary
make build

# Install globally
sudo mv ./bin/grepai /usr/local/bin/

# Verify installation
grepai version
```

**Requirements:**
- Go 1.24 or later
- Make (optional, can use `go build` directly)

## Verifying Installation

After installation, verify everything works:

```bash
# Check version
grepai version
# Output: grepai version 0.24.0

# Check available commands
grepai --help
```

## Updating GrepAI

### Check for Updates

```bash
grepai update --check
```

### Perform Update

```bash
# Auto-update (downloads and installs latest)
grepai update

# Or via Homebrew
brew upgrade grepai
```

## Uninstalling

### Homebrew

```bash
brew uninstall grepai
```

### Manual

```bash
# Remove binary
sudo rm /usr/local/bin/grepai

# Optionally remove config directories
rm -rf ~/.grepai
```

## Next Steps

After installation:
1. **Install Ollama** for local embeddings (see `grepai-ollama-setup` skill)
2. **Initialize a project** with `grepai init`
3. **Start indexing** with `grepai watch`

## Common Installation Issues

❌ **Problem:** `command not found: grepai`
✅ **Solution:** Ensure `/usr/local/bin` is in your PATH:
```bash
export PATH="$PATH:/usr/local/bin"
```

❌ **Problem:** Permission denied during installation
✅ **Solution:** Use `sudo` for the installation command or install to a user directory

❌ **Problem:** Homebrew tap not found
✅ **Solution:** Add the tap first:
```bash
brew tap yoanbernabeu/tap
brew install grepai
```

## Output Format

After successful installation:

```
✅ GrepAI installed successfully
   Version: 0.24.0
   Location: /usr/local/bin/grepai

Next steps:
1. Install Ollama: brew install ollama
2. Initialize project: grepai init
3. Start indexing: grepai watch
```
