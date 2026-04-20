---
title: "Group Policy"
type: concept
sources:
  - raw/articles/Groups.md
  - raw/articles/WhatsApp.md
  - "raw/articles/Group Messages.md"
related:
  - "[[pairing|Pairing]]"
  - "[[ai-agent-gateway|AI Agent Gateway]]"
tags:
  - openclaw
  - access-control
  - group-chat
  - mention-gating
  - session-isolation
  - activation
created: 2026-04-20
updated: 2026-04-20
---

# Group Policy

## Definition
Group Policy is OpenClaw's unified access-control model for group chats, applied consistently across nine channels (Discord, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, Zalo). It evaluates inbound group messages through three independent gates — `groupPolicy`, group allowlists, and mention gating — and separates **trigger authorization** (who may activate the agent) from **context visibility** (what supplemental history reaches the model) (source: [[groups-source|Groups]]).

## Key Points
- **Three-gate evaluation order.** `groupPolicy` (open/disabled/allowlist) → group allowlists (`groups`, `groupAllowFrom`, channel-specific) → mention gating (`requireMention`, `/activation`). Failing an earlier gate short-circuits later ones (source: [[groups-source|Groups]])
- **Fail-closed default.** `groupPolicy: "allowlist"` is the default, and an empty allowlist blocks all group messages. When a provider block is entirely missing, policy falls back to `allowlist` rather than inheriting `channels.defaults.groupPolicy` (source: [[groups-source|Groups]])
- **Trigger authorization ≠ context visibility.** Allowlists primarily gate who can fire the agent; they are not (yet) a universal redaction boundary for quoted, replied, or forwarded context. Some channels (Slack thread seeding, Matrix reply/thread lookups) already filter supplemental context by sender; others pass it through as received. A planned `contextVisibility` setting will unify this with `all` / `allowlist` / `allowlist_quote` modes (source: [[groups-source|Groups]])
- **DM pairing never admits groups.** DM pairing approvals (`*-allowFrom` stores) unlock DMs only; group authorization must come from config allowlists like `groupAllowFrom` or the documented per-channel fallback. This is the load-bearing invariant separating DM and group trust (source: [[groups-source|Groups]])
- **Mention gating is the default reply filter.** Group messages require an @mention unless overridden per group. Replying to or quoting a bot message counts as an implicit mention on channels that expose that metadata (Telegram, WhatsApp, Slack, Discord, Microsoft Teams, ZaloUser). `mentionPatterns` provide a case-insensitive regex fallback; unsafe nested-repetition patterns are dropped (source: [[groups-source|Groups]])
- **`mentionPatterns` is cross-channel.** Originally WhatsApp-specific, `agents.list[].groupChat.mentionPatterns` is now consulted by Telegram, Discord, Slack, and iMessage as well. `messages.groupChat.mentionPatterns` is a global fallback (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Two activation modes (WhatsApp).** `mention` (default) requires an explicit ping — `mentionedJids`, a `mentionPatterns` match, or the bot's E.164 in text. `always` wakes the agent on every message but expects self-gating: the agent returns the exact silent token `NO_REPLY` / `no_reply` when it has nothing worth saying (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Pending-only history injection.** When mention gating silences a message, it is still buffered as context. On the next triggered turn, up to `historyLimit` pending messages are wrapped and injected; set `0` to disable. Global default via `messages.groupChat.historyLimit`, overrides per channel/account (source: [[groups-source|Groups]])
- **Context injection markers (WhatsApp).** Pending context wraps as `[Chat messages since your last reply - for context]`; the triggering message wraps as `[Current message - respond to this]`; every batch ends with `[from: Sender Name (+E164)]`. Ephemeral/view-once wrappers are stripped before text/mention extraction so pings inside them still trigger (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Group system prompt blurb.** On the first turn of a group session (and on every `/activation` change), the system prompt gets a short blurb naming the group subject, members, activation mode, and reminding the agent to address the specific sender (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Per-channel allowlist shapes.** WhatsApp / Telegram / Signal / iMessage / Microsoft Teams / Zalo use `groupAllowFrom`; Discord uses `guilds.<id>.channels`; Slack uses `channels.*`; Matrix uses `groups` (prefer room IDs or aliases — name lookup is best-effort) plus optional per-room `users` allowlists. Group DMs are controlled via separate `dm.*` blocks where applicable (source: [[groups-source|Groups]])
- **Session-key isolation enables split execution.** Groups always use non-main session keys (`agent:<agentId>:<channel>:group:<id>`) while DMs typically use the main session. With `sandbox.mode: "non-main"`, a single agent can run DMs on-host with full tools while groups execute sandboxed with restricted tools. Telegram forum topics append `:topic:<threadId>` so each topic is its own session (source: [[groups-source|Groups]])
- **Session-scoped standalone commands.** `/verbose on`, `/trace on`, `/think high`, `/new` or `/reset`, `/compact`, and `/status` are scoped to the group's session — they do not leak into the agent's personal DM session. The session store lives at `~/.openclaw/agents/<agentId>/sessions/sessions.json` (WhatsApp); a missing entry just means the group hasn't triggered a run yet (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Echo suppression & typing indicators (WhatsApp).** Echo suppression keys on the combined batch string — identical text twice without mentions only produces one response. Typing indicators in groups follow `agents.defaults.typingMode` (default: `message` when unmentioned) (source: [[group-messages-source|Group Messages (WhatsApp)]])
- **Per-group tool restrictions compose, deny wins.** Channel configs accept `tools` and `toolsBySender` (with `id:`, `e164:`, `username:`, `name:`, `"*"` prefixes) scoped to a specific group/room/channel. Resolution is most-specific-first, but these layer *on top of* global/agent tool policy — any deny at any layer wins (source: [[groups-source|Groups]])
- **Owner-only activation toggle.** `/activation mention` and `/activation always` let the owner flip per-group mention gating at runtime. Owner is `channels.whatsapp.allowFrom` (or the bot's self E.164). Currently honored only on WhatsApp; other surfaces ignore the command (source: [[groups-source|Groups]])
- **Context fields on inbound payloads.** Group messages carry `ChatType=group`, `GroupSubject`, `GroupMembers`, and `WasMentioned` (mention gating result). Telegram forum topics add `MessageThreadId` and `IsForum`. BlueBubbles can optionally enrich unnamed macOS group participants from the local Contacts database after gating passes (source: [[groups-source|Groups]])
- **Default WhatsApp composition.** WhatsApp's default is `groupPolicy: "allowlist"` combined with `groups: { "*": { requireMention: true } }` — groups must be explicitly listed, and even then only mentions trigger a reply. Duplicate JSON5 keys silently shadow earlier entries (source: [[whatsapp-source|WhatsApp]])

## Sources
- [[groups-source|Groups]] — canonical description of the unified group model and evaluation order
- [[whatsapp-source|WhatsApp]] — concrete instantiation of Group Policy on a single channel
- [[group-messages-source|Group Messages (WhatsApp)]] — activation modes, session commands, and context-marker format on WhatsApp

## Related Concepts
- [[pairing|Pairing]] — complementary owner-approval gate for DMs and devices; deliberately scoped to not include groups
- [[ai-agent-gateway|AI Agent Gateway]] — the architectural pattern whose shared session and session-key isolation make cross-channel group policy possible
