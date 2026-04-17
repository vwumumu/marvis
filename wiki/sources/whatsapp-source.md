---
title: "WhatsApp (Web channel)"
type: source
sources:
  - raw/articles/WhatsApp.md
source_path: raw/articles/WhatsApp.md
tags:
  - openclaw
  - whatsapp
  - chat-channel
  - baileys
created: 2026-04-17
updated: 2026-04-17
---

# WhatsApp (Web channel)

## Key Takeaways
- OpenClaw's WhatsApp integration is production-ready and uses the WhatsApp Web protocol via Baileys; Gateway owns linked session(s) (source: [[whatsapp-source|WhatsApp]])
- Installation is on-demand via `openclaw onboard` or `openclaw channels add --channel whatsapp`, pulling `@openclaw/whatsapp` from npm on stable/beta (source: [[whatsapp-source|WhatsApp]])
- A dedicated WhatsApp number is recommended over personal-number mode for cleaner routing and DM allowlists (source: [[whatsapp-source|WhatsApp]])
- DM access control uses `dmPolicy` with four modes: `pairing` (default), `allowlist`, `open`, `disabled`; `allowFrom` takes E.164 numbers (source: [[whatsapp-source|WhatsApp]])
- Multi-account support with per-account overrides for policies, credentials, media limits, and reaction levels (source: [[whatsapp-source|WhatsApp]])
- Text chunking defaults to 4000 chars with `length` or `newline` mode; media supports image/video/audio/document with a 50MB default cap and auto-optimization for images (source: [[whatsapp-source|WhatsApp]])
- Reaction behavior is controlled by `reactionLevel` (off/ack/minimal/extensive), defaulting to `minimal` (source: [[whatsapp-source|WhatsApp]])
- Bun runtime is flagged as incompatible for stable WhatsApp gateway operation — use Node (source: [[whatsapp-source|WhatsApp]])

## Detailed Notes

### Runtime model
The Gateway owns the WhatsApp socket and reconnect loop. Outbound sends require an active WhatsApp listener for the target account. Status and broadcast chats (`@status`, `@broadcast`) are ignored. Direct chats use DM session rules; group sessions are isolated per-agent/per-group. The transport honors standard proxy environment variables (`HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY`).

### Deployment patterns
- **Dedicated number (recommended):** separate WhatsApp identity for OpenClaw with clearer DM allowlists and routing boundaries
- **Personal-number fallback:** onboarding writes a self-chat-friendly baseline (`dmPolicy: "allowlist"`, `selfChatMode: true`, includes personal number in `allowFrom`)
- Self-chat safeguards skip read receipts for self-chat turns and ignore mention-JID auto-trigger that would ping yourself

### Access control
- `dmPolicy` modes: `pairing` (default), `allowlist`, `open` (requires `allowFrom` to include `"*"`), `disabled`
- Pairings are persisted in channel allow-store and merged with configured `allowFrom`
- If no allowlist is configured, the linked self number is allowed by default
- Outbound `fromMe` DMs are never auto-paired
- Multi-account overrides: `channels.whatsapp.accounts.<id>.dmPolicy` takes precedence over channel defaults

### Message normalization
Quoted replies are wrapped with `[Replying to <sender> id:<stanzaId>] ... [/Replying]` markers. Reply metadata fields (`ReplyToId`, `ReplyToBody`, `ReplyToSender`) are populated when available. For groups, unprocessed messages can be buffered and injected as context when the bot is triggered (default `historyLimit: 50`, set `0` to disable). Injection markers: `[Chat messages since your last reply - for context]` and `[Current message - respond to this]`.

### Delivery and media
- Text chunking: `textChunkLimit = 4000`, `chunkMode = "length" | "newline"` (newline mode prefers paragraph boundaries)
- Media: image, video, audio (PTT voice-note), document; `audio/ogg` is rewritten to `audio/ogg; codecs=opus`; `gifPlayback: true` for animated GIFs
- Captions apply to the first media item when sending multi-media payloads
- Media sources: HTTP(S), `file://`, or local paths
- Size caps: `mediaMaxMb` default 50 (both inbound and outbound); images auto-optimized to fit limits
- On media send failure, first-item fallback sends a text warning instead of silent drop

### Credentials and multi-account
- Current auth path: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- Backup: `creds.json.bak`
- Legacy default auth in `~/.openclaw/credentials/` still recognized/migrated for default-account flows
- Default account selection: `default` if present, otherwise first configured account id (sorted)

### Reactions
`channels.whatsapp.reactionLevel` controls reaction breadth:

| Level | Ack | Agent-initiated | Description |
|---|---|---|---|
| `off` | No | No | No reactions |
| `ack` | Yes | No | Pre-reply receipt only |
| `minimal` | Yes | Yes (conservative) | Default |
| `extensive` | Yes | Yes (encouraged) | Broad agent use |

Ack reactions (`channels.whatsapp.ackReaction`) fire immediately on inbound accept; group mode options: `always | mentions | never`.

### Troubleshooting
- Not linked → `openclaw channels login --channel whatsapp`
- Reconnect loop → `openclaw doctor`, `openclaw logs --follow`
- Group messages ignored → check `groupPolicy`, `groupAllowFrom`, `groups` allowlist, mention gating, and duplicate JSON5 keys (later entries override earlier ones)

## Related Pages
- [[openclaw|OpenClaw]] — the gateway that hosts the WhatsApp channel
- [[whatsapp-channel|WhatsApp Channel]] — channel-specific entity details
- [[baileys|Baileys]] — the WhatsApp Web library used for integration
- [[pairing|Pairing]] — owner-approval flow behind the default `pairing` DM policy
