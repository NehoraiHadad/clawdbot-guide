---
name: Clawdbot Guide
description: This skill should be used when the user mentions "clawdbot", "moltbot", "molt.bot", "lobster bot", asks about "channel configuration", "WhatsApp setup", "Telegram setup", "Discord setup", "Slack setup", "iMessage setup", "Signal setup", needs help with "exec tool", "browser tool", "session tools", "sub-agents", "multi-agent", "cron jobs", "webhooks", asks about "moltbot.json", "clawdbot configuration", "gateway config", "agent routing", "sandbox", "skills", "ClawdHub", or any Moltbot/Clawdbot platform question. Provides comprehensive knowledge of the Moltbot/Clawdbot AI assistant platform.
version: 0.1.0
---

# Clawdbot/Moltbot Complete Guide

## IMPORTANT: Always Verify with Current Documentation

**Moltbot/Clawdbot is a rapidly evolving platform.** This skill may contain outdated information.

**Before implementing any solution:**
1. **Fetch the current documentation** for the specific topic from https://docs.molt.bot/
2. Use WebFetch on the relevant docs page to get the latest information
3. The LLM-friendly reference at https://docs.molt.bot/llms.txt contains the full documentation index

**Common documentation URLs:**
- Channels: `https://docs.molt.bot/channels/<channel-name>` (e.g., `/channels/whatsapp`)
- Tools: `https://docs.molt.bot/tools/<tool-name>` (e.g., `/tools/exec`, `/tools/browser`)
- CLI: `https://docs.molt.bot/cli/<command>` (e.g., `/cli/sessions`, `/cli/cron`)
- Concepts: `https://docs.molt.bot/concepts/<topic>` (e.g., `/concepts/multi-agent`)

---

## IMPORTANT: Continuous Learning Instruction

**When working with Clawdbot and discovering new information:**

1. **Immediately update this skill** with any new knowledge discovered during usage
2. Update the relevant reference file in `~/.claude/skills/clawdbot-guide/references/`
3. Add new sections for undocumented features
4. Correct any inaccurate information
5. Add practical examples from real usage
6. **Note what changed** - if the docs differ from this skill, update the skill

This skill improves through use. Every Clawdbot interaction is an opportunity to enhance this knowledge base.

---

## Overview

**Moltbot** (also known as Clawdbot) is a self-hosted AI assistant platform that operates through a local gateway architecture. It enables interaction with Claude or other AI models through multiple communication channels.

**Key characteristics:**
- Local-first gateway serving as unified control plane
- WebSocket-based architecture coordinating sessions, channels, tools, and events
- Multi-channel support (WhatsApp, Telegram, Discord, Slack, iMessage, Signal, etc.)
- Isolated agent workspaces with flexible routing
- Skills system for extensibility
- Voice integration with Wake and Talk modes

**Installation:**
```bash
npm install -g moltbot@latest
moltbot onboard
```

Requires Node.js 22+. Installs as daemon service (launchd on macOS, systemd on Linux).

## Quick Reference

### Configuration Location

Primary config: `~/.clawdbot/moltbot.json`

### Common Commands

| Command | Purpose |
|---------|---------|
| `moltbot doctor` | Run diagnostics and repairs |
| `moltbot sessions` | List conversation sessions |
| `moltbot channels login` | Set up channel authentication |
| `moltbot cron` | Manage scheduled jobs |
| `moltbot browser` | Control browser automation |
| `moltbot status` | Check gateway status |

### Dashboard Access

Local dashboard: `http://127.0.0.1:18789/`

## Tool Categories

### Core Execution Tools

The `exec` tool runs shell commands with configurable security.

**Key parameters:**
- `command` - Shell command to execute
- `host` - Execution location: `sandbox` (default), `gateway`, or `node`
- `security` - Enforcement: `deny`, `allowlist`, `full`
- `elevated` - Request elevated permissions

**Reference:** See `references/tools-core.md` for complete exec tool documentation.

### Browser Automation

Agent-controlled browser with two modes:
- **Clawd (Managed):** Dedicated browser instance with CDP control
- **Chrome (Extension Relay):** Controls existing Chrome tabs via extension

**Reference:** See `references/tools-browser.md` for browser automation details.

### Session & Sub-agents

- **Sessions:** Isolated conversation contexts per channel/group
- **Sub-agents:** Background agent runs for parallel tasks

**Reference:** See `references/tools-sessions.md` for session management.

### Automation

- **Cron:** Scheduled jobs via gateway scheduler
- **Webhooks:** HTTP endpoints for external triggers
- **Polling:** Periodic checks for external events

**Reference:** See `references/tools-automation.md` for automation setup.

## Supported Channels

### Messaging Platforms

| Channel | Auth Method | Key Config |
|---------|-------------|------------|
| WhatsApp | QR code via Baileys | `channels.whatsapp` |
| Telegram | BotFather token | `channels.telegram.botToken` |
| Discord | Bot token | `channels.discord.token` |
| Slack | App + Bot tokens | `channels.slack` |
| Signal | signal-cli | `channels.signal` |

**Reference:** See `references/channels-messaging.md` for detailed setup.

### Apple Platforms

iMessage integration via `imsg` CLI tool (macOS only).

**Reference:** See `references/channels-apple.md` for iMessage setup.

## Agent Architecture

### Workspace Structure

Each agent maintains:
- Dedicated workspace directory
- State directory (`agentDir`)
- Session store
- Bootstrap files (AGENTS.md, SOUL.md, USER.md, etc.)

### Bootstrap Files

| File | Purpose |
|------|---------|
| AGENTS.md | Operating instructions and memory |
| SOUL.md | Persona, boundaries, tone |
| USER.md | User profile and preferences |
| TOOLS.md | User-maintained tool guidance |
| IDENTITY.md | Agent name and identity |
| BOOTSTRAP.md | One-time initialization (auto-deleted) |

**Reference:** See `references/agent-architecture.md` for agent details.

### Multi-Agent Routing

Routing uses deterministic, most-specific-wins matching:
1. Direct peer matches (DM/group/channel ID)
2. Guild ID (Discord)
3. Team ID (Slack)
4. Account ID matching
5. Channel-wide fallback
6. Default agent assignment

**Reference:** See `references/agent-architecture.md` for routing rules.

## Configuration

### moltbot.json Structure

```json5
{
  agents: {
    defaults: {
      workspace: "~/workspace",
      model: "anthropic/claude-opus-4-5",
      sandbox: { mode: "off" }
    },
    list: [{ id: "main", /* agent config */ }]
  },
  channels: {
    whatsapp: { enabled: true, /* ... */ },
    telegram: { enabled: true, botToken: "..." }
  },
  tools: {
    exec: { pathPrepend: "/custom/bin" },
    elevated: { enabled: true, allowFrom: [...] }
  },
  skills: {
    entries: { "skill-name": { enabled: true } }
  }
}
```

**Reference:** See `references/configuration.md` for complete config reference.

## Skills System

Skills extend agent capabilities through SKILL.md files.

**Loading hierarchy (highest to lowest):**
1. Workspace skills: `<workspace>/skills`
2. Managed skills: `~/.clawdbot/skills`
3. Bundled skills: shipped with installation

**ClawdHub:** Public skills marketplace at https://clawdhub.com

```bash
clawdhub install <skill-slug>
clawdhub update --all
```

**Reference:** See `references/skills-system.md` for skill development.

## Memory System

### Daily Logs

Stored in `memory/YYYY-MM-DD.md` - append-only notes loaded at session start.

### Long-term Memory

`MEMORY.md` contains curated durable facts, loaded only in private sessions.

### Vector Search

Optional semantic search across markdown files using embeddings.

**Reference:** See `references/memory-sessions.md` for memory details.

## Common Patterns

### Basic WhatsApp Setup

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "pairing",
      selfChatMode: false
    }
  }
}
```

Then run: `moltbot channels login`

### Enable Sandboxing for Groups

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "non-main" }
    }
  }
}
```

### Configure Elevated Mode

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: ["+15551234567"]
    }
  }
}
```

## Custom Scripts Location

Custom scripts for operations not supported by Moltbot's API can be placed in the workspace scripts folder:

```
<workspace>/scripts/
└── whatsapp/
    ├── list-groups.mjs      # List all WhatsApp groups
    ├── filter-groups.mjs    # Filter groups by keywords
    └── passive-monitor.mjs  # Monitor messages passively
```

**Default location:** `~/clawd/scripts/whatsapp/` (or wherever `agents.defaults.workspace` points)

These scripts use direct Baileys access for operations like:
- Listing all WhatsApp groups (not available via Moltbot API)
- Filtering groups by name/size
- Passive message monitoring without agent response

See `references/baileys-direct-access.md` for script documentation and available Baileys functions.

## Known Limitations & Workarounds

### WhatsApp Passive Monitoring

**Problem:** No built-in way to receive/log WhatsApp messages without the agent responding.

| Policy Setting | Messages Logged? | Agent Responds? |
|----------------|------------------|-----------------|
| `groupPolicy: "open"` | Yes | Yes (on mention) |
| `groupPolicy: "allowlist"` (empty) | No | No |
| `groupPolicy: "disabled"` | No | No |

**Workaround:** Use direct Baileys access for passive monitoring. See `references/baileys-direct-access.md`.

### WhatsApp Group Discovery

**Problem:** `clawdbot directory groups list` only reads from config, not live WhatsApp data.

**Workaround:** Use the Baileys scripts in the workspace:
```bash
# List all groups (briefly disconnects Moltbot)
node ~/clawd/scripts/whatsapp/list-groups.mjs personal

# Filter by keywords (uses cached data, no reconnect)
node ~/clawd/scripts/whatsapp/filter-groups.mjs personal --cached --filter גן,בית\ ספר
node ~/clawd/scripts/whatsapp/filter-groups.mjs personal --cached --filter work,עבודה
node ~/clawd/scripts/whatsapp/filter-groups.mjs personal --cached --min-participants 50
```

See `references/baileys-direct-access.md` for full documentation.

### Telegram vs WhatsApp Action Gating

Telegram supports `actions.sendMessage: false` to prevent sending messages while still receiving. WhatsApp does NOT have this feature.

## Troubleshooting

### Run Diagnostics

```bash
moltbot doctor --deep
moltbot doctor --repair
```

### Check Gateway Status

```bash
moltbot status
moltbot health
```

### Debug Mode

```bash
moltbot --debug
```

## Additional Resources

### Reference Files

Detailed documentation in `references/`:

- **`tools-core.md`** - exec tool, security modes, background processes
- **`tools-browser.md`** - browser automation, CDP, snapshots
- **`tools-sessions.md`** - sessions, sub-agents, multi-agent
- **`tools-automation.md`** - cron, webhooks, polling
- **`channels-messaging.md`** - WhatsApp, Telegram, Discord, Slack, Signal
- **`channels-apple.md`** - iMessage configuration
- **`configuration.md`** - moltbot.json, bootstrap files
- **`commands-reference.md`** - CLI commands
- **`skills-system.md`** - skills, ClawdHub
- **`agent-architecture.md`** - agents, routing, sandboxing
- **`memory-sessions.md`** - memory, session management
- **`baileys-direct-access.md`** - Direct WhatsApp/Baileys access for advanced operations

### External Documentation

- **Official docs:** https://docs.molt.bot/
- **LLM reference:** https://docs.molt.bot/llms.txt
- **GitHub:** https://github.com/moltbot/moltbot

---

## Continuous Improvement

**CRITICAL:** This skill must be kept current. Moltbot is under active development and changes frequently.

**Workflow for every Clawdbot task:**

1. **Check current docs first** - WebFetch the relevant page from https://docs.molt.bot/ before relying solely on this skill
2. **Compare with this skill** - Note any differences between current docs and this skill
3. **Update immediately** - If the docs have new or different information:
   - Update the relevant reference file in `~/.claude/skills/clawdbot-guide/references/`
   - Add new sections for newly discovered features
   - Correct any outdated or inaccurate information
   - Add practical examples from real usage
4. **Learn from errors** - If something doesn't work as documented here, fetch the latest docs and update

Every interaction with Clawdbot is an opportunity to improve this knowledge base. Treat discrepancies between this skill and official docs as bugs to fix immediately.
