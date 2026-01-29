# Configuration Reference

## Overview

Primary configuration: `~/.clawdbot/moltbot.json`

Format: **JSON5** (supports comments and trailing commas)

**Important:** Moltbot refuses startup when configs contain unknown keys, malformed values, or invalid types. This prevents silent failures.

## Validation & Safe Practices

### Before Making Changes

```bash
moltbot doctor          # Check for issues
moltbot doctor --fix    # Auto-repair and backup config
moltbot status          # Verify credentials loaded
moltbot models status   # Confirm API keys present
```

### Common Mistakes to Avoid

| Mistake | Solution |
|---------|----------|
| Missing `gateway.mode` | Set `moltbot config set gateway.mode local` |
| Per-agent auth issues | Run onboarding per agent or copy `auth-profiles.json` |
| Non-loopback without auth | Set `gateway.auth.token` and `gateway.auth.mode` |
| Deprecated keys | Use `gateway.auth.token` not `gateway.token` |
| Unsupported models | Check `moltbot models list` |

### Testing Changes

Always test with ad-hoc gateway runs using `--verbose` before committing to service supervisor.

---

## Complete Schema

### Top-Level Keys

```json5
{
  gateway: {},      // Server & auth
  agents: {},       // Agent configuration
  models: {},       // Model providers
  channels: {},     // Messaging channels
  session: {},      // Session management
  messages: {},     // Message handling
  tools: {},        // Tool configuration
  auth: {},         // Auth profiles
  env: {},          // Environment
  web: {},          // Web features
  browser: {},      // Browser automation
  commands: {},     // Command settings
  talk: {},         // Voice/TTS
  skills: {},       // Skills system
  plugins: {},      // Plugins
  logging: {},      // Logging
  cron: {},         // Scheduled jobs
  wizard: {}        // Setup wizard state
}
```

---

## gateway Configuration

```json5
{
  gateway: {
    mode: "local",              // Required: local|remote
    port: 18789,                // WebSocket port
    bind: "loopback",           // loopback|lan|tailnet|custom
    auth: {
      token: "your-token",      // Required if bind != loopback
      mode: "token"             // token|none
    }
  }
}
```

---

## agents Configuration

### agents.defaults

```json5
{
  agents: {
    defaults: {
      // Workspace
      workspace: "~/workspace",           // Required
      repoRoot: "~/repos",
      skipBootstrap: false,
      bootstrapMaxChars: 10000,
      userTimezone: "America/Los_Angeles",
      timeFormat: "auto",                 // auto|12|24

      // Models
      model: "anthropic/claude-opus-4-5", // or {primary, fallbacks}
      imageModel: { primary: "...", fallbacks: [] },
      thinkingDefault: "off",             // high|low|off
      verboseDefault: "off",
      elevatedDefault: "off",

      // Execution
      timeoutSeconds: 300,
      mediaMaxMb: 5,
      maxConcurrent: 1,

      // Sandbox
      sandbox: {
        mode: "off",                      // off|non-main|all
        scope: "session",                 // session|agent|shared
        workspaceAccess: "rw",            // none|ro|rw
        workspaceRoot: "~/sandboxed",
        docker: {
          image: "moltbot/sandbox:latest",
          containerPrefix: "moltbot-",
          workdir: "/workspace",
          readOnlyRoot: true,
          network: "none",
          memory: "512m",
          cpus: "1.0",
          setupCommand: "apt-get update && apt-get install -y jq"
        },
        browser: {
          enabled: false,
          allowHostControl: false
        }
      },

      // Context
      contextTokens: 200000,
      contextPruning: {
        mode: "adaptive",                 // off|adaptive|aggressive
        keepLastAssistants: 3,
        softTrimRatio: 0.8,
        hardClearRatio: 0.95
      },

      // Compaction
      compaction: {
        mode: "default",                  // default|safeguard
        reserveTokensFloor: 10000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 1000
        }
      },

      // Streaming
      blockStreamingDefault: "off",
      blockStreamingBreak: "text_end",    // text_end|message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },

      // UI
      humanDelay: { mode: "off" },        // off|natural|custom
      typingMode: "thinking",             // never|instant|thinking|message
      typingIntervalSeconds: 5,

      // Sub-agents
      subagents: {
        model: "anthropic/claude-sonnet-4",
        maxConcurrent: 8,
        archiveAfterMinutes: 60
      },

      // Heartbeat
      heartbeat: {
        every: "3600",                    // seconds or duration string
        model: "anthropic/claude-sonnet-4",
        session: "heartbeat",
        target: "whatsapp",
        to: "+15551234567"
      }
    }
  }
}
```

### agents.list[]

```json5
{
  agents: {
    list: [
      {
        id: "main",                       // Required, unique
        default: true,
        name: "Main Agent",
        workspace: "~/main-workspace",    // Override defaults
        agentDir: "~/.clawdbot/agents/main",
        model: "anthropic/claude-opus-4-5",

        identity: {
          name: "Clawd",
          theme: "orange",
          emoji: "🦞"
        },

        groupChat: {
          mentionPatterns: ["@clawd", "hey clawd"]
        },

        sandbox: { mode: "off" },         // Per-agent override

        subagents: {
          allowAgents: ["research", "ops"] // or ["*"]
        },

        tools: {
          profile: "full",
          allow: ["read", "write"],
          deny: ["gateway"],
          elevated: { enabled: true }
        }
      }
    ]
  }
}
```

---

## session Configuration

```json5
{
  session: {
    scope: "per-sender",                  // per-sender|per-channel|per-peer|global
    dmScope: "main",                      // main|per-peer|per-channel-peer|per-account-channel-peer
    mainKey: "main",

    identityLinks: {
      "john": ["+15551234567", "discord:123456789"]
    },

    reset: {
      mode: "daily",                      // daily|idle
      atHour: 4,                          // 4 AM
      idleMinutes: 120
    },

    resetByType: {
      dm: { mode: "daily", atHour: 4 },
      group: { mode: "idle", idleMinutes: 120 },
      thread: { mode: "idle", idleMinutes: 60 }
    },

    resetTriggers: ["/new", "/reset"],
    heartbeatIdleMinutes: 60,
    store: "~/.clawdbot/sessions",

    agentToAgent: { maxPingPongTurns: 5 },

    sendPolicy: {
      rules: [
        { action: "deny", match: { chatType: "group" } }
      ],
      default: "allow"
    }
  }
}
```

---

## messages Configuration

```json5
{
  messages: {
    responsePrefix: "auto",               // string|auto
    ackReaction: "👀",
    ackReactionScope: "group-mentions",   // group-mentions|group-all|direct|all
    removeAckAfterReply: true,

    groupChat: { historyLimit: 50 },

    queue: {
      mode: "collect",                    // collect|steer|followup|interrupt|steer-backlog
      debounceMs: 500,
      cap: 10,
      drop: "old",                        // old|new|summarize
      byChannel: { whatsapp: "steer" }
    },

    inbound: {
      debounceMs: 100,
      byChannel: { telegram: 200 }
    },

    tts: {
      auto: "off",                        // off|always|inbound|tagged
      mode: "final",                      // final|all
      provider: "elevenlabs",             // elevenlabs|openai
      maxTextLength: 4000,
      timeoutMs: 30000,
      elevenlabs: {
        apiKey: "${ELEVENLABS_API_KEY}",
        voiceId: "...",
        modelId: "eleven_turbo_v2_5"
      }
    }
  }
}
```

---

## tools Configuration

```json5
{
  tools: {
    profile: "full",                      // minimal|coding|messaging|full
    allow: ["read", "write", "exec"],
    deny: ["gateway"],

    agentToAgent: { enabled: false, allow: ["main", "research"] },

    byProvider: {
      "anthropic/claude-haiku-3": {
        profile: "minimal",
        deny: ["exec"]
      }
    },

    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15551234567"],
        telegram: ["123456789", "@username"],
        discord: ["user_id"]
      }
    },

    web: {
      search: {
        enabled: true,
        maxResults: 10,
        timeoutSeconds: 30
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        timeoutSeconds: 30
      }
    },

    media: {
      concurrency: 2,
      image: { enabled: true, maxBytes: 5000000 },
      audio: { enabled: true, maxBytes: 10000000 },
      video: { enabled: true, maxBytes: 50000000 }
    },

    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      notifyOnExit: true,
      applyPatch: { enabled: false }
    },

    sandbox: {
      tools: {
        allow: ["read", "write"],
        deny: ["exec", "gateway"]
      }
    }
  }
}
```

---

## channels Configuration

### Common Pattern

```json5
{
  channels: {
    defaults: {
      groupPolicy: "allowlist"            // open|allowlist|disabled
    },

    // Per-channel configs below
  }
}
```

### WhatsApp

```json5
{
  channels: {
    whatsapp: {
      enabled: true,
      dmPolicy: "pairing",                // pairing|allowlist|open|disabled
      allowFrom: ["+15551234567"],
      selfChatMode: false,
      sendReadReceipts: true,
      textChunkLimit: 4000,
      chunkMode: "length",                // length|newline
      mediaMaxMb: 50,
      historyLimit: 50,
      dmHistoryLimit: 100,
      configWrites: true,

      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions"                 // always|mentions|never
      },

      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: {
        "jid@g.us": {
          allowFrom: ["+15551234567"],
          mention: true
        }
      },

      accounts: {
        "work": {
          authDir: "~/.clawdbot/credentials/whatsapp/work",
          sendReadReceipts: false
        }
      }
    }
  }
}
```

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",                // or TELEGRAM_BOT_TOKEN env
      dmPolicy: "pairing",
      allowFrom: ["123456789", "@username"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["987654321"],
      textChunkLimit: 4000,
      mediaMaxMb: 5,
      historyLimit: 50,
      linkPreview: true,
      replyToMode: "first",               // off|first|all
      streamMode: "partial",              // off|partial|block
      reactionNotifications: "own",       // off|own|all

      groups: {
        "-1001234567890": {
          requireMention: false,
          skills: ["search"],
          systemPrompt: "Extra instructions",
          topics: {
            "456": { requireMention: false }
          }
        },
        "*": { requireMention: true }
      },

      capabilities: {
        inlineButtons: "allowlist"        // off|dm|group|all|allowlist
      },

      actions: {
        reactions: true,
        sendMessage: true,
        deleteMessage: true,
        sticker: false
      },

      customCommands: [
        { command: "backup", description: "Git backup" }
      ],

      // Webhook mode (optional)
      webhookUrl: "https://example.com/telegram",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook"
    }
  }
}
```

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",            // or DISCORD_BOT_TOKEN env
      textChunkLimit: 2000,
      maxLinesPerMessage: 17,
      mediaMaxMb: 8,
      historyLimit: 20,
      allowBots: false,
      replyToMode: "off",

      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["user_id"],
        groupEnabled: false
      },

      groupPolicy: "allowlist",

      guilds: {
        "guild_id": {
          users: ["user_id"],
          requireMention: true,
          reactionNotifications: "own",
          channels: {
            "channel_id": {
              allow: true,
              requireMention: false,
              users: ["user_id"],
              skills: ["search"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      },

      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        messages: true,
        threads: true,
        pins: true,
        memberInfo: true,
        roles: false,
        moderation: false
      }
    }
  }
}
```

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",                     // socket|http
      appToken: "xapp-...",
      botToken: "xoxb-...",
      userToken: "xoxp-...",              // optional
      userTokenReadOnly: true,
      dmPolicy: "pairing",
      groupPolicy: "allowlist",
      textChunkLimit: 4000,
      mediaMaxMb: 20,
      replyToMode: "off",

      replyToModeByChatType: {
        direct: "off",
        group: "off",
        channel: "first"
      },

      // HTTP mode only
      signingSecret: "...",
      webhookPath: "/slack/events"
    }
  }
}
```

---

## skills Configuration

```json5
{
  skills: {
    allowBundled: ["skill1", "skill2"],   // optional allowlist

    load: {
      watch: true,
      watchDebounceMs: 250,
      extraDirs: ["/custom/skills"]
    },

    install: {
      nodeManager: "npm"                  // npm|pnpm|yarn|bun
    },

    entries: {
      "skill-name": {
        enabled: true,
        apiKey: "SECRET",
        env: { VAR: "value" },
        config: { customKey: "value" }
      }
    }
  }
}
```

---

## browser Configuration

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: false,               // Security: disable arbitrary JS
    defaultProfile: "clawd",
    remoteCdpTimeoutMs: 1500,
    remoteCdpHandshakeTimeoutMs: 3000,

    profiles: {
      clawd: {
        cdpPort: 18800,
        color: "#FF4500",
        headless: false
      },
      remote: {
        cdpUrl: "https://browserless.io?token=KEY"
      }
    },

    snapshotDefaults: {
      mode: "efficient"
    }
  }
}
```

---

## cron Configuration

```json5
{
  cron: {
    enabled: true,
    store: "~/.clawdbot/cron/jobs.json",
    maxConcurrentRuns: 1
  }
}
```

---

## logging Configuration

```json5
{
  logging: {
    level: "info",                        // debug|info|warn|error
    file: "~/.clawdbot/logs/moltbot.log",
    consoleLevel: "info",
    consoleStyle: "pretty"                // pretty|json
  }
}
```

---

## Type References

| Type | Format | Example |
|------|--------|---------|
| Duration | String | `"30m"`, `"1h"`, `"500ms"` |
| E.164 | Phone number | `"+15551234567"` |
| Env reference | `${VAR}` | `"${ANTHROPIC_API_KEY}"` |
| File path | Tilde expansion | `"~/workspace"` |

---

## Bootstrap Files

Six files in workspace directory injected on session start:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Operating instructions and memory |
| `SOUL.md` | Persona, boundaries, tone |
| `USER.md` | User profile and preferences |
| `TOOLS.md` | Tool guidance (conventions, not control) |
| `IDENTITY.md` | Agent name, vibe, emoji |
| `BOOTSTRAP.md` | One-time init (auto-deleted) |

**Note:** Blank files skipped. Large files trimmed with marker.

---

## Continuous Improvement

**Always verify with current docs:** Before implementing, fetch the relevant page from https://docs.molt.bot/gateway/configuration to check for updates.

When using Clawdbot and discovering undocumented features, corrections, or better practices:
1. Update this file at `~/.claude/skills/clawdbot-guide/references/configuration.md`
2. Add new sections for newly discovered features
3. Correct any outdated or inaccurate information
4. Add practical examples from real usage
