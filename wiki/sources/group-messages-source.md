---
title: "Group Messages (WhatsApp)"
type: source
sources:
  - "raw/articles/Group Messages.md"
source_path: "raw/articles/Group Messages.md"
tags:
  - openclaw
  - whatsapp
  - group-chat
  - activation
  - session-isolation
  - context-injection
created: 2026-04-20
updated: 2026-04-20
---

# Group Messages (WhatsApp)

## Key Takeaways
- WhatsApp groups use two activation modes: `mention` (default, requires an @-mention, regex pattern match, or the bot's E.164 in text) and `always` (wake on every message, but reply only when valuable — return `NO_REPLY` / `no_reply` to stay silent) (source: [[group-messages-source|Group Messages (WhatsApp)]])
- `channels.whatsapp.groups` doubles as both per-group activation defaults and an allowlist — include `"*"` to allow all groups (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Per-group sessions use `agent:<agentId>:whatsapp:group:<jid>` keys; session-scoped commands (`/verbose on`, `/trace on`, `/think high`, `/new`, `/reset`, `/compact`, `/status`) apply only to the group session, never touching the personal DM session (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Pending (mention-skipped) messages are injected as `[Chat messages since your last reply - for context]` and the triggering message as `[Current message - respond to this]`; already-in-session messages are not re-injected (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Every group batch ends with `[from: Sender Name (+E164)]` so the agent knows who is speaking (source: [[group-messages-source|Group Messages (WhatsApp)]])
- The `mentionPatterns` setting — originally WhatsApp-only — is now used by Telegram, Discord, Slack, and iMessage as well; per-agent patterns live under `agents.list[].groupChat.mentionPatterns`, with `messages.groupChat.mentionPatterns` as a global fallback (source: [[group-messages-source|Group Messages (WhatsApp)]])
- The group system prompt includes a short blurb on the first turn (and whenever `/activation` changes) naming the group, members, activation mode, and reminding the agent to address the specific sender (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Ephemeral and view-once messages are unwrapped before text/mention extraction, so pings inside them still trigger (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Echo suppression keys on the combined batch string: sending identical text twice without mentions will only get one response (source: [[group-messages-source|Group Messages (WhatsApp)]])
- Heartbeats are skipped for group threads, and typing indicators in groups follow `agents.defaults.typingMode` (default: `message` when unmentioned) (source: [[group-messages-source|Group Messages (WhatsApp)]])

## Detailed Notes

### Activation modes
- **`mention` (default):** agent wakes only when pinged. Pings can be real WhatsApp @-mentions (`mentionedJids`), matches against `mentionPatterns` regexes, or the bot's E.164 appearing anywhere in the text.
- **`always`:** agent receives every message but must self-gate, replying only when it can add meaningful value. To stay silent it returns the exact token `NO_REPLY` or `no_reply`.
- Defaults live in `channels.whatsapp.groups`; owners can override per group at runtime via `/activation mention` or `/activation always`. Only the owner (from `channels.whatsapp.allowFrom`, or the bot's own E.164 if unset) can change activation. Standalone `/status` in the group reports the current mode.

### Per-group sessions
- Session key shape: `agent:<agentId>:whatsapp:group:<jid>`
- Personal DM state is completely independent of any group session.
- Session-scoped standalone commands: `/verbose on`, `/trace on`, `/think high`, `/new` or `/reset`, `/compact`, `/status`.
- Heartbeats are skipped for group threads (no noisy broadcasts).
- Session store location: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (default). A missing entry simply means the group hasn't triggered a run yet.

### Context injection format
- Pending-only group messages (default 50, configurable via `historyLimit`) are wrapped under `[Chat messages since your last reply - for context]`.
- The triggering message is wrapped under `[Current message - respond to this]`.
- Messages already in the session are not re-injected.
- Every batch ends with `[from: Sender Name (+E164)]` so the agent can address the right sender.
- Ephemeral and view-once messages are unwrapped before extracting text and mentions.

### Group system prompt blurb
On the first turn of a group session (and on every `/activation` change) the system prompt gets a short injection like:
```
You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.
```
When group metadata isn't available, the agent is still told it is in a group chat.

### Mention patterns and multi-channel scope
- `agents.list[].groupChat.mentionPatterns` per-agent regexes are case-insensitive and pass the same safe-regex guardrails used elsewhere (invalid and unsafe-nested-repetition patterns are ignored).
- `messages.groupChat.mentionPatterns` serves as a global fallback.
- Originally WhatsApp-specific, these patterns are now consulted by Telegram, Discord, Slack, and iMessage as well.
- WhatsApp still sends canonical mentions via `mentionedJids` when someone taps the contact — the number/name fallback is rarely needed but is a safety net for when WhatsApp strips the visual `@` in text.

### Group policy and allowlist
- `channels.whatsapp.groupPolicy` accepts `open | disabled | allowlist`; default is `allowlist` (blocked until you add senders).
- `channels.whatsapp.groupAllowFrom` is the allowlist source (with explicit `channels.whatsapp.allowFrom` as fallback).
- Setting `channels.whatsapp.groups` simultaneously provides per-group defaults *and* functions as an allowlist — include `"*"` to allow all groups.

### Operational notes
- **Echo suppression:** keyed on the combined batch string. Identical text twice without mentions → only the first response fires.
- **Typing indicators:** follow `agents.defaults.typingMode` (default: `message` when unmentioned).
- **Verification:** run the gateway with `--verbose` to see `inbound web message` log lines showing `from: <groupJid>` and the `[from: …]` suffix.

### How to use (quickstart)
1. Add the OpenClaw-linked WhatsApp account to the group.
2. Mention it: `@openclaw …` (or include its number). Only allowlisted senders can trigger unless `groupPolicy: "open"` is set.
3. The agent prompt includes recent group context plus the trailing `[from: …]` marker so it can address the right person.
4. Send session-level directives (`/verbose on`, `/trace on`, `/think high`, `/new`, `/reset`, `/compact`) as standalone messages — they apply only to that group's session.

## Related Pages
- [[whatsapp-channel|WhatsApp Channel]] — the channel this document specifies
- [[group-policy|Group Policy]] — cross-channel group access model WhatsApp inherits
- [[groups-source|Groups]] — generalized group model referenced by this WhatsApp-specific doc
- [[openclaw|OpenClaw]] — the gateway that owns the session store and activation machinery
- [[pairing|Pairing]] — DM-only owner-approval boundary that does not extend to group access
