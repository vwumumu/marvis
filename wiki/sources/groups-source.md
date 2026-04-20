---
title: "Groups"
type: source
sources:
  - raw/articles/Groups.md
source_path: raw/articles/Groups.md
tags:
  - openclaw
  - group-chat
  - access-control
  - mention-gating
  - session-isolation
created: 2026-04-20
updated: 2026-04-20
---

# Groups

## Key Takeaways
- OpenClaw applies a consistent group-chat model across nine surfaces: Discord, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, and Zalo (source: [[groups-source|Groups]])
- Group access is a three-step evaluation: `groupPolicy` (open/disabled/allowlist) → group allowlists (`groups`, `groupAllowFrom`, channel-specific) → mention gating (`requireMention`, `/activation`) (source: [[groups-source|Groups]])
- Default behavior is `groupPolicy: "allowlist"` with `requireMention: true` — an empty allowlist blocks all group messages, and replies require an @mention (source: [[groups-source|Groups]])
- DM pairing approval never grants group access; group authorization is always explicit via group allowlists (source: [[groups-source|Groups]])
- Trigger authorization (who can activate the agent) is separate from context visibility (what history/quote/forward context reaches the model) — the latter is currently channel-specific, with planned `contextVisibility` modes (`all` / `allowlist` / `allowlist_quote`) (source: [[groups-source|Groups]])
- Group messages always land in non-main session keys (`agent:<agentId>:<channel>:group:<id>`), while DMs typically land in the main session — enabling single-agent patterns like "DMs on host, groups sandboxed" via `sandbox.mode: "non-main"` (source: [[groups-source|Groups]])
- Telegram forum topics extend the session key with `:topic:<threadId>` so each topic has its own session; heartbeats are skipped for group sessions (source: [[groups-source|Groups]])
- Per-group tool restrictions (`tools`, `toolsBySender` with explicit key prefixes like `id:`, `e164:`, `username:`, `name:`) compose with global/agent tool policy — deny still wins (source: [[groups-source|Groups]])
- Owner-only `/activation mention|always` command toggles per-group mention gating; owner is determined by `channels.whatsapp.allowFrom` (or the bot's self E.164) and is currently only honored by WhatsApp (source: [[groups-source|Groups]])
- Group inbound payloads set `ChatType=group`, `GroupSubject`, `GroupMembers`, and `WasMentioned`; the agent system prompt injects a group intro on the first turn of a new group session (source: [[groups-source|Groups]])

## Detailed Notes

### The two-plane model: trigger authorization vs context visibility
OpenClaw draws a sharp line between **who may trigger the agent** (gated by `groupPolicy` + allowlists + mention gating) and **what supplemental context is injected** (reply text, quotes, thread history, forwarded metadata). Allowlists primarily govern triggering; they are not a universal redaction boundary. Current behavior is channel-specific: Slack thread seeding and Matrix reply/thread lookups already filter supplemental context by sender, while other channels pass quote/reply/forward context through as received. A planned `contextVisibility` setting will unify this with three modes: `all` (default, as-received), `allowlist` (filter supplemental context to allowlisted senders), and `allowlist_quote` (allowlist plus one explicit quote/reply exception).

### Evaluation order for group messages
The group-message flow is:
1. `groupPolicy` — if `disabled`, drop; if `allowlist` and the group is not allowed, drop
2. Group allowlists — `*.groups`, `*.groupAllowFrom`, and channel-specific allowlists (Discord `guilds.<id>.channels`, Slack `channels`, Matrix `groups`)
3. Mention gating — if `requireMention: true` and no mention is detected, store the message as context only (not a reply trigger)

### `groupPolicy` modes
- `"open"` — groups bypass allowlists, but mention gating still applies
- `"disabled"` — block all group messages entirely
- `"allowlist"` — only allow groups/rooms that match the configured allowlist; **this is the default**, and an empty allowlist blocks everything
- Runtime safety: when a provider block is completely missing (`channels.<provider>` absent), group policy falls back to a fail-closed mode (typically `allowlist`) instead of inheriting `channels.defaults.groupPolicy`

### Per-channel allowlist shapes
- WhatsApp / Telegram / Signal / iMessage / Microsoft Teams / Zalo: use `groupAllowFrom` (with explicit `allowFrom` fallback)
- Discord: `channels.discord.guilds.<id>.channels`
- Slack: `channels.slack.channels`
- Matrix: `channels.matrix.groups` (prefer room IDs or aliases; joined-room name lookup is best-effort, unresolved names ignored at runtime); `channels.matrix.groupAllowFrom` for sender restriction; per-room `users` allowlists also supported
- Telegram allowlist keys: user IDs (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) or usernames (`"@alice"` or `"alice"`); prefixes are case-insensitive
- Group DMs are controlled separately (`channels.discord.dm.*`, `channels.slack.dm.*`)

### DM pairing vs group authorization
DM pairing approvals (`*-allowFrom` store entries) apply only to DM access. Group commands still require explicit group sender authorization from config allowlists such as `groupAllowFrom` or the documented per-channel fallback. This is the load-bearing invariant: approving someone via `openclaw pairing approve <channel> <CODE>` unlocks DMs but not groups.

### Mention gating
Group messages require a mention by default unless overridden per group. Defaults live per subsystem under `*.groups."*"`.
- Replying to a bot message counts as an implicit mention when the channel supports reply metadata (Telegram, WhatsApp, Slack, Discord, Microsoft Teams, ZaloUser)
- Quoting a bot message can also count as an implicit mention on channels that expose quote metadata
- `mentionPatterns` are case-insensitive safe regex patterns; invalid patterns and unsafe nested-repetition forms are ignored
- Surfaces that provide explicit mentions still pass; patterns are only a fallback
- Per-agent override: `agents.list[].groupChat.mentionPatterns` (useful when multiple agents share a group)
- Mention gating is only enforced when mention detection is possible (native mentions or `mentionPatterns` are configured)
- Discord defaults live in `channels.discord.guilds."*"` (overridable per guild/channel)
- Group history context is wrapped uniformly and is **pending-only** (messages skipped due to mention gating); global default via `messages.groupChat.historyLimit`, overrides via `channels.<channel>.historyLimit` (or `channels.<channel>.accounts.*.historyLimit`); set `0` to disable

### Session keys and sandboxing
- Group sessions: `agent:<agentId>:<channel>:group:<id>`
- Rooms/channels: `agent:<agentId>:<channel>:channel:<id>`
- Telegram forum topics: append `:topic:<threadId>` so each topic is its own session
- DMs: main session (or per-sender if configured)
- Heartbeats are skipped for group sessions

This naming enables a "personal DMs + public groups, single agent" pattern: DMs hit `agent:main:main` while groups use non-main keys, so `sandbox.mode: "non-main"` runs groups in Docker while DMs stay on-host. One agent brain (shared workspace + memory), two execution postures. For truly separate workspaces/personas, use a second agent with bindings instead.

Example split (groups sandboxed to messaging-only tools):
```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

### Per-group tool restrictions
Channel configs can restrict tools per group/room/channel:
- `tools`: allow/deny for the whole group
- `toolsBySender`: per-sender overrides with explicit key prefixes — `id:<senderId>`, `e164:<phone>`, `username:<handle>`, `name:<displayName>`, `"*"` wildcard. Legacy unprefixed keys still accepted but matched as `id:` only.

Resolution order (most specific wins):
1. Group/channel `toolsBySender` match
2. Group/channel `tools`
3. Default (`"*"`) `toolsBySender` match
4. Default (`"*"`) `tools`

These compose **in addition to** global/agent tool policy — deny always wins. Nesting differs per channel (Discord `guilds.*.channels.*`, Slack `channels.*`, Microsoft Teams `teams.*.channels.*`).

### Activation command (owner-only)
Group owners can toggle per-group activation with `/activation mention` or `/activation always`. Owner is determined by `channels.whatsapp.allowFrom` (or the bot's self E.164 when unset). Send the command as a standalone message. Other surfaces currently ignore `/activation`.

### Display labels
- UI labels use `displayName` when available, formatted as `<channel>:<token>`
- `#room` is reserved for rooms/channels
- Group chats use `g-<slug>` (lowercase, spaces → `-`, keep `#@+._-`)

### Context fields on group payloads
Inbound group messages set:
- `ChatType=group`
- `GroupSubject` (if known)
- `GroupMembers` (if known)
- `WasMentioned` (mention gating result)
- Telegram forum topics also include `MessageThreadId` and `IsForum`

BlueBubbles can optionally enrich unnamed macOS group participants from the local Contacts database before populating `GroupMembers`; off by default, runs only after normal group gating passes.

The agent system prompt includes a group intro on the first turn of a new group session, reminding the model to respond like a human, avoid Markdown tables, minimize empty lines, follow normal chat spacing, and avoid typing literal `\n` sequences.

### iMessage specifics
- Prefer `chat_id:<id>` when routing or allowlisting
- List chats: `imsg chats --limit 20`
- Group replies always go back to the same `chat_id`

### WhatsApp specifics
Detailed WhatsApp-only group behavior (history injection, mention handling details) lives under the channel's Group messages page, not this document.

## Related Pages
- [[openclaw|OpenClaw]] — the gateway that applies this group model uniformly across channels
- [[group-policy|Group Policy]] — the unified access-control concept distilled from this document
- [[whatsapp-channel|WhatsApp Channel]] — one surface where this model is in force
- [[pairing|Pairing]] — DM pairing approvals never extend to groups
- [[ai-agent-gateway|AI Agent Gateway]] — architectural pattern this model reinforces
