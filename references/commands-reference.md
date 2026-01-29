# CLI Commands Reference

## Command Overview

Moltbot provides 50+ CLI commands for managing the gateway, channels, sessions, and tools.

## Core Commands

### Gateway Management

| Command | Description |
|---------|-------------|
| `moltbot start` | Start the gateway daemon |
| `moltbot stop` | Stop the gateway daemon |
| `moltbot restart` | Restart the gateway |
| `moltbot status` | Check gateway status |
| `moltbot health` | Health check endpoint |

### Setup & Configuration

| Command | Description |
|---------|-------------|
| `moltbot onboard` | Run setup wizard |
| `moltbot setup` | Initialize configuration |
| `moltbot configure` | Edit configuration |
| `moltbot update` | Update to latest version |
| `moltbot uninstall` | Remove Moltbot |

### Diagnostics

| Command | Description |
|---------|-------------|
| `moltbot doctor` | Run diagnostics |
| `moltbot doctor --repair` | Apply fixes, backup config |
| `moltbot doctor --deep` | Comprehensive checks |
| `moltbot logs` | View gateway logs |

## Channel Commands

### Authentication

```bash
# Login to channel
moltbot channels login
moltbot channels login --channel telegram
moltbot channels login --account secondary

# List channels
moltbot channels

# Channel status
moltbot channels status
```

### Pairing

```bash
# List pending approvals
moltbot pairing

# Approve sender
moltbot pairing approve whatsapp <code>
moltbot pairing approve telegram <code>

# Deny sender
moltbot pairing deny <code>
```

## Session Commands

### List Sessions

```bash
# Basic listing
moltbot sessions

# Active sessions (within N minutes)
moltbot sessions --active 120

# JSON output
moltbot sessions --json
```

### In-Chat Commands

| Command | Description |
|---------|-------------|
| `/status` | Show session status |
| `/context list` | List context items |
| `/new` | Start new session |
| `/reset` | Reset current session |

## Cron Commands

```bash
# List cron jobs
moltbot cron

# Show job details
moltbot cron show <job-id>

# Edit job delivery
moltbot cron edit <job-id> --deliver --channel telegram --to "123456789"

# Disable delivery
moltbot cron edit <job-id> --no-deliver
```

## Browser Commands

### Navigation

```bash
# Open URL
moltbot browser open https://example.com

# Take snapshot
moltbot browser snapshot
moltbot browser snapshot --interactive
moltbot browser snapshot --mode role

# Screenshot
moltbot browser screenshot
moltbot browser screenshot --full-page
```

### Interaction

```bash
# Click element
moltbot browser click e12
moltbot browser click e12 --double

# Type text
moltbot browser type e23 "text"
moltbot browser type e23 "query" --submit

# Drag and drop
moltbot browser drag e10 e11
```

### State

```bash
# Cookies
moltbot browser cookies set name value --url "https://example.com"
moltbot browser cookies get --url "https://example.com"

# Geolocation
moltbot browser set geo 37.7749 -122.4194

# Timezone
moltbot browser set timezone America/New_York
```

## Agent Commands

```bash
# List agents
moltbot agents

# Agent info
moltbot agent <agent-id>

# Switch agent
moltbot agent use <agent-id>
```

## Memory Commands

```bash
# View memory
moltbot memory

# Search memory
moltbot memory search "query"
```

## Node Commands

```bash
# List nodes
moltbot nodes

# Node status
moltbot nodes status

# Pair node
moltbot nodes pair
```

## Skills Commands

```bash
# List skills
moltbot skills

# Skill info
moltbot skills show <skill-name>

# Enable/disable skill
moltbot skills enable <skill-name>
moltbot skills disable <skill-name>
```

## ClawdHub Commands

```bash
# Install skill
clawdhub install <skill-slug>

# Update all skills
clawdhub update --all

# Sync skills
clawdhub sync --all

# Search skills
clawdhub search "keyword"
```

## In-Chat Slash Commands

### Session Control

| Command | Description |
|---------|-------------|
| `/new` | Start new session |
| `/reset` | Reset session |
| `/status` | Show status |
| `/context list` | List context |

### Mode Control

| Command | Description |
|---------|-------------|
| `/elevated on` | Enable elevated mode |
| `/elevated full` | Full elevated (auto-approve) |
| `/elevated off` | Disable elevated |
| `/exec` | Toggle exec defaults |

### Sub-agents

| Command | Description |
|---------|-------------|
| `/subagents` | List sub-agents |
| `/subagents list` | List active |
| `/subagents stop <id>` | Stop sub-agent |
| `/subagents send <id> <msg>` | Send message |

### Process Control

| Command | Description |
|---------|-------------|
| `/process` | List background processes |
| `/process poll <id>` | Check process status |
| `/process send-keys <id>` | Send input |
| `/process submit <id>` | Submit command |

### Configuration

| Command | Description |
|---------|-------------|
| `/config` | Show configuration |
| `/activation` | Set activation mode |

## Debug Commands

```bash
# Enable debug mode
moltbot --debug

# Verbose logging
moltbot --verbose

# TUI interface
moltbot tui
```

## Useful Command Patterns

### Quick Health Check

```bash
moltbot status && moltbot health
```

### Reset Channel Auth

```bash
moltbot channels login --channel whatsapp --force
```

### Export Session Data

```bash
moltbot sessions --json > sessions.json
```

### Debug Gateway Issues

```bash
moltbot doctor --deep
moltbot logs --tail 100
```

---

## Continuous Improvement

**Always verify with current docs:** Before implementing, fetch the relevant page from https://docs.molt.bot/ to check for updates.


When using Clawdbot and discovering undocumented features, corrections, or better practices:
1. Update this file at `~/.claude/skills/clawdbot-guide/references/commands-reference.md`
2. Add new sections for newly discovered features
3. Correct any outdated or inaccurate information
4. Add practical examples from real usage
