# Migration Guide: v1.x to v2.0

This guide helps you migrate from Agentic Studio v1.x (Go-based installer) to v2.0 (Node.js npm package).

## Overview

**Agentic Studio v1.x (v1.15.0 and earlier)** was distributed as a Go binary with an interactive TUI installer that:
- Installed agents and commands to `~/.claude/`
- Installed templates and rules to `.agentic-studio/` (default) or a custom path
- Copied the binary to `.agentic-studio/bin/agentic-studio`
- Modified `~/.claude/settings.json` to add permissions, hooks, and statusLine
- Created a lock file at `.agentic-studio/agentic-studio.lock`

**Version 2.0** is distributed as an npm package with TypeScript/Node.js implementation and Ink-based TUI installer.

## Migration Steps

### Step 1: Uninstall v1.x

#### 1. Remove Claude Assets

```bash
# Remove agents from ~/.claude/agents/
# Note: v1.15.0 used nested directory structure
rm -rf ~/.claude/agents/the-*

# Remove commands from ~/.claude/commands/
rm -rf ~/.claude/commands/s

# Remove output style
rm -f ~/.claude/output-styles/agentic-studio.md
```

#### 2. Remove Startup Assets

```bash
# Default installation path (or check your custom path)
# Remove templates
rm -rf ./config/agentic-studio

# OR for project-local installs
rm -rf ./.agentic-studio

##### 3. Clean Up settings.json

The v1.x installer modified `~/.claude/settings.json`. You need to remove startup-managed entries:

**Open `~/.claude/settings.json` and remove:**

1. **permissions.additionalDirectories** - Remove entries containing `.agentic-studio`:
```json
{
  "permissions": {
    "additionalDirectories": [
      "~/.agentic-studio"  // <<- REMOVE THIS
    ]
  }
}
```

2. **statusLine** - Remove if it references agentic-studio binary:
```json
{
  "statusLine": {"  // <-- REMOVE THIS ENTIRE SECTION
    "type": "command",
    "command": "~/.agentic-studio/bin/agentic-studio statusline
  }
}
```

3. **outputStyle** - Remove if set to "The Startup":
```json
{
  "outputStyle": "The Startup"  // REMOVE OR CHANGE TO "Default"
}
```

4. **hooks** - v1.x did NOT install hooks in settings.json, but if present, remove any with `"_source": "agentic-studio"` or commands containing `"agentic-studio"`

**Important:** Only remove Agentic Studio-related entries. Keep all other settings intact.

##### 4. Clean Up settings.local.json (if exists)

If `~/.claude/settings.local.json` exists, apply the same cleanup as settings.json above.

#### Verify Complete Removal

```bash
# Check for any remaining files in .claude
find ~/.claude -name "*startup*" -o -name "*agentic-studio*"

If `find` returns no results and `.agentic-studio` doesn't exist (or is empty), cleanup is complete.

### Step 2: Install v2.x

Check the README.md for installation instructions of v2.x
