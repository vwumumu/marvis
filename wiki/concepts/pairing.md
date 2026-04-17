---
title: "Pairing"
type: concept
sources:
  - raw/articles/Pairing.md
  - raw/articles/WhatsApp.md
  - "raw/articles/Chat Channels.md"
related:
  - "[[ai-agent-gateway|AI Agent Gateway]]"
tags:
  - openclaw
  - access-control
  - pairing
  - devices
  - security
created: 2026-04-17
updated: 2026-04-17
---

# Pairing

## Definition
In OpenClaw, pairing is the explicit owner-approval gate that admits trust into the gateway. It applies in two contexts: DM pairing (which senders may message the bot on a given channel) and node device pairing (which devices/nodes may join the gateway network). In both cases the owner must approve a pending request before access is granted (source: [[pairing-source|Pairing]]).

## Key Points
- **Two contexts, one mental model.** DM pairing admits *senders*; node pairing admits *devices* with `role: node`. The owner-approval gate is the same idea applied to different subjects (source: [[pairing-source|Pairing]])
- **DM pairing code UX.** 8 uppercase chars, ambiguous chars (`0O1I`) removed, 1-hour expiry, max 3 pending per channel; the bot only mints a new code when a new request is created, so at most ~once per hour per sender (source: [[pairing-source|Pairing]])
- **CLI approval.** `openclaw pairing list <channel>` / `openclaw pairing approve <channel> <CODE>` for DMs; `openclaw devices list/approve/reject` for nodes (source: [[pairing-source|Pairing]])
- **DM policy integration.** `pairing` is one of four channel `dmPolicy` modes (alongside `allowlist`, `open`, `disabled`) — the default on WhatsApp (source: [[whatsapp-source|WhatsApp]])
- **Broad channel support.** DM pairing works across 20+ channels including bluebubbles, discord, imessage, matrix, msteams, signal, slack, telegram, whatsapp, openclaw-weixin, and zalo (source: [[pairing-source|Pairing]])
- **Scope boundary: DMs only.** Approving a DM pairing code never grants group access — group authorization is a separate store using per-channel constructs like `groupAllowFrom`, `groups`, or per-group overrides (source: [[pairing-source|Pairing]])
- **Account-scoped DM storage.** DM pending requests live in `~/.openclaw/credentials/<channel>-pairing.json`; approved allowlists live in `<channel>-allowFrom.json` for the default account and `<channel>-<accountId>-allowFrom.json` for non-default accounts (source: [[pairing-source|Pairing]])
- **Setup code = bootstrap secret.** Node pairing setup codes are base64-encoded JSON containing a WebSocket URL and a short-lived `bootstrapToken` — to be treated like a password while valid (source: [[pairing-source|Pairing]])
- **Role-prefixed scope checks.** Bootstrap scope validation is role-prefixed: operator scopes (`operator.approvals`, `operator.read`, `operator.talk.secrets`, `operator.write`) only satisfy operator requests; non-operator roles must request scopes under their own role prefix (source: [[pairing-source|Pairing]])
- **Pairing record is the source of truth.** Active device tokens stay bounded to the approved role set in the pairing record — stray tokens outside the approved roles do not create new access (source: [[pairing-source|Pairing]])
- **Retry semantics.** If a device retries pairing with different auth details, the previous pending request is superseded and a new `requestId` is issued (source: [[pairing-source|Pairing]])
- **Channel linking ≠ sender pairing.** WhatsApp's QR pairing (via Baileys) links the Gateway's own account; it is distinct from DM pairing, which admits external senders to an already-linked account (source: [[chat-channels-source|Chat Channels]])

## Sources
- [[pairing-source|Pairing]] — canonical description of DM and device pairing
- [[whatsapp-source|WhatsApp]] — pairing as the default DM policy on the WhatsApp channel
- [[chat-channels-source|Chat Channels]] — channel-level DM pairing and allowlist enforcement

## Related Concepts
- [[ai-agent-gateway|AI Agent Gateway]] — pairing is the owner-trust boundary of a self-hosted gateway
