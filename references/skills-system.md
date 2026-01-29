# Skills System

## Overview

Moltbot uses AgentSkills-compatible skill folders to extend agent capabilities. Each skill is a directory containing a `SKILL.md` file with YAML frontmatter and instructions.

## SKILL.md Format

### Required Fields

```yaml
---
name: skill-identifier
description: Brief functionality description
---

# Skill content here
```

**Important:** The parser supports **single-line** frontmatter keys only. Metadata must be a single-line JSON object.

### Optional Frontmatter

| Parameter | Values | Purpose |
|-----------|--------|---------|
| `homepage` | URL | Shown as "Website" in macOS Skills UI |
| `user-invocable` | true/false (default: true) | Exposes skill as slash command |
| `disable-model-invocation` | true/false (default: false) | Excludes from model prompt |
| `command-dispatch` | tool | Bypasses model, dispatches directly |
| `command-tool` | tool name | Specifies tool for dispatch |
| `command-arg-mode` | raw (default) | Forwards raw args to tool |

### Metadata for Gating

```yaml
metadata: {"moltbot":{"requires":{"bins":["jq"],"env":["API_KEY"],"config":["browser.enabled"]},"os":["darwin"]}}
```

## Gating & Load-Time Filtering

Skills filter via `metadata.moltbot` JSON object:

### Availability Controls

| Field | Description |
|-------|-------------|
| `always: true` | Include skill regardless of other gates |
| `os` | List of eligible platforms: `darwin`, `linux`, `win32` |

### Dependency Requirements

| Field | Description |
|-------|-------------|
| `requires.bins` | All listed binaries must exist on PATH |
| `requires.anyBins` | At least one binary must exist |
| `requires.env` | Environment variables (checked in config or process) |
| `requires.config` | Paths in moltbot.json that must be truthy |

### Additional Metadata

| Field | Description |
|-------|-------------|
| `emoji` | Used by macOS Skills UI |
| `primaryEnv` | Associates env variable with `skills.entries.<name>.apiKey` |

## Skill Loading Hierarchy

Three locations with hierarchical precedence (highest to lowest):

1. **Workspace skills** (`<workspace>/skills`)
2. **Managed/local skills** (`~/.clawdbot/skills`)
3. **Bundled skills** (shipped with install)

**Additional layers:**
- Plugin skills participate in normal precedence
- Extra directories via `skills.load.extraDirs` have lowest precedence

**Multi-agent context:**
- Per-agent skills in workspace
- Shared skills in managed/local directory
- Name conflicts resolved by precedence hierarchy

## Installation & Installers

Skills can include installer specs for automated setup.

### Installer Kinds

| Kind | Description |
|------|-------------|
| `brew` | Homebrew formula |
| `node` | npm/pnpm/yarn/bun package |
| `go` | Go module |
| `uv` | Python uv package |
| `download` | Direct file download |

### Installer Metadata

```yaml
metadata: {"moltbot":{"installers":[{"id":"brew-jq","kind":"brew","bins":["jq"],"label":"jq via Homebrew"}]}}
```

**Download installer options:** `url`, `archive` (tar.gz/tar.bz2/zip), `stripComponents`, `targetDir`

## Configuration

### Per-Skill Settings

```json5
{
  skills: {
    entries: {
      "skill-name": {
        enabled: true,        // false disables permanently
        apiKey: "SECRET",     // maps to primaryEnv
        env: { VAR: "value" }, // inject if not already set
        config: { key: "value" }
      }
    }
  }
}
```

### Loading Settings

```json5
{
  skills: {
    load: {
      watch: true,           // hot reload on SKILL.md changes
      watchDebounceMs: 250,
      extraDirs: ["/path/to/skills"]
    },
    install: {
      nodeManager: "npm"     // npm/pnpm/yarn/bun
    },
    allowBundled: ["skill1", "skill2"]  // optional allowlist
  }
}
```

## Environment Injection

Per agent run:
1. Read skill metadata requirements
2. Apply `skills.entries.<key>.env` or `apiKey` to process environment
3. Build system prompt with eligible skills
4. Restore original environment after run

**Scope:** Agent run only (not global shell environment)

**Security:** Secrets injected into host process for that turn; keep secrets from prompts/logs.

## Performance & Runtime

### Session Snapshots

- Eligible skills captured when session starts
- Reused across subsequent turns
- Changes take effect on new session

### Skills Watcher

- Monitors skill folders for SKILL.md changes
- Auto-refreshes eligible skills list (hot reload)
- Configured via `skills.load.watch`

### Token Cost

Skills add deterministic overhead to prompts:

**Formula:** 195 base chars + Σ(97 + escaped_name + escaped_description + escaped_location)

**Estimates:**
- Base overhead: ~49 tokens
- Per skill: ~24 tokens plus field lengths
- XML escaping expands `& < > " '`

## ClawdHub Registry

Public skills marketplace: https://clawdhub.com

### Commands

```bash
clawdhub install <skill-slug>   # Install to ./skills
clawdhub update --all           # Update all installed
clawdhub sync --all             # Scan and publish updates
clawdhub search "keyword"       # Search registry
```

Default: installs to current directory's `./skills`

## Plugin Skills

Plugins can ship skills by listing directories in `moltbot.plugin.json`:

```json5
{
  skills: ["skills/my-skill", "skills/another-skill"]
}
```

Plugin skills participate in normal precedence and can be gated via config requirements.

## CLI Commands

```bash
moltbot skills                     # List all skills
moltbot skills show <skill-name>   # Show details
moltbot skills enable <skill-name>
moltbot skills disable <skill-name>
```

## Security Considerations

- Treat third-party skills as **trusted code** — review before enabling
- Prefer sandboxed runs for untrusted inputs
- Binary requirements checked on host at load time
- Sandboxed agents need binaries in container via `setupCommand`
- Secrets from `skills.entries.*.env` and `*.apiKey` injected into host process

## Creating Skills

### Directory Structure

```
skill-name/
├── SKILL.md           # Required
├── scripts/           # Optional executables
├── references/        # Optional documentation
└── assets/            # Optional files for output
```

### Minimal Example

```yaml
---
name: my-skill
description: Does something useful
---

# My Skill

Instructions for using this skill.
```

### With Gating

```yaml
---
name: browser-skill
description: Browser automation helpers
metadata: {"moltbot":{"requires":{"bins":["chromium"],"config":["browser.enabled"]}}}
---

# Browser Skill

Requires browser enabled and Chromium installed.
```

### Use `{baseDir}` for Paths

Reference skill folder paths with `{baseDir}` placeholder.

---

## Continuous Improvement

**Always verify with current docs:** Before implementing, fetch the relevant page from https://docs.molt.bot/tools/skills to check for updates.

When using Clawdbot and discovering undocumented features, corrections, or better practices:
1. Update this file at `~/.claude/skills/clawdbot-guide/references/skills-system.md`
2. Add new sections for newly discovered features
3. Correct any outdated or inaccurate information
4. Add practical examples from real usage
