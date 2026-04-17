---
title: "Pairing"
type: source
sources:
  - raw/articles/Pairing.md
source_path: raw/articles/Pairing.md
tags:
  - openclaw
  - pairing
  - access-control
  - devices
  - security
created: 2026-04-17
updated: 2026-04-17
---

# Pairing

## Key Takeaways
- Pairing is OpenClaw's explicit owner-approval step, applied in two distinct places: DM pairing (who may message the bot) and node device pairing (which devices may join the gateway) (source: [[pairing-source|Pairing]])
- DM pairing codes are 8 uppercase chars with ambiguous chars (`0O1I`) excluded, expire after 1 hour, and are capped at 3 pending per channel by default (source: [[pairing-source|Pairing]])
- The bot only issues a new pairing message when a new request is created — approximately once per hour per sender (source: [[pairing-source|Pairing]])
- DM pairing is supported across 20+ channels including bluebubbles, discord, imessage, matrix, msteams, signal, slack, telegram, whatsapp, openclaw-weixin, and zalo (source: [[pairing-source|Pairing]])
- DM pairing state lives in `~/.openclaw/credentials/`; approved allowlist files are account-scoped for non-default accounts (`<channel>-<accountId>-allowFrom.json`) but unscoped for the default account (source: [[pairing-source|Pairing]])
- DM approval never implies group access — group authorization requires explicit per-channel config like `groupAllowFrom`, `groups`, or per-group overrides (source: [[pairing-source|Pairing]])
- Node devices connect with `role: node` and require gateway-side approval; Telegram `/pair` (via the `device-pair` plugin) is the recommended iOS flow (source: [[pairing-source|Pairing]])
- Setup codes are base64-encoded JSON containing `url` (Gateway WebSocket URL) and `bootstrapToken` (short-lived bootstrap token); treat them like a password while valid (source: [[pairing-source|Pairing]])
- Bootstrap scope checks are role-prefixed: operator scopes only satisfy operator requests, non-operator roles must request scopes under their own role prefix (source: [[pairing-source|Pairing]])
- The pairing record is the durable source of truth for approved roles — stray token entries outside the approved role set do not create new access (source: [[pairing-source|Pairing]])

## Detailed Notes

### DM pairing (inbound chat access)
When a channel's `dmPolicy` is set to `pairing`, unknown senders receive a short pairing code and their message is held unprocessed until the owner approves. Codes are 8 uppercase characters with ambiguous characters (`0`, `O`, `1`, `I`) removed, expire after 1 hour, and the bot only sends a new pairing message when a new request is created (so at most roughly once per hour per sender). Pending requests cap at 3 per channel by default — further requests are silently ignored until one expires or is approved.

Approval workflow:

```shellscript
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Supported channels: `bluebubbles`, `discord`, `feishu`, `googlechat`, `imessage`, `irc`, `line`, `matrix`, `mattermost`, `msteams`, `nextcloud-talk`, `nostr`, `openclaw-weixin`, `signal`, `slack`, `synology-chat`, `telegram`, `twitch`, `whatsapp`, `zalo`, `zalouser`.

**State files** in `~/.openclaw/credentials/`:
- Pending requests: `<channel>-pairing.json`
- Approved allowlist (default account): `<channel>-allowFrom.json`
- Approved allowlist (non-default account): `<channel>-<accountId>-allowFrom.json`

Account scoping: non-default accounts read/write only their scoped allowlist file; the default account uses the channel-scoped unscoped file. These files are sensitive — they gate access to the assistant.

**Scope boundary:** DM approval is for direct-message access only. Group authorization is a separate store and approving a DM pairing code does not allow that sender to run group commands or control the bot in groups. Group access must be configured explicitly via `groupAllowFrom`, `groups`, or per-group/per-topic overrides depending on the channel.

### Node device pairing (iOS / Android / macOS / headless)
Nodes connect to the Gateway as devices with `role: node`. The Gateway creates a device pairing request that must be approved.

**Telegram-driven flow (recommended for iOS)** uses the `device-pair` plugin:
1. Message `/pair` to the bot in Telegram
2. The bot replies with an instruction message and a separate setup-code message (copy-paste friendly)
3. Open the OpenClaw iOS app → Settings → Gateway and paste the setup code
4. In Telegram: `/pair pending` (review request IDs, role, and scopes), then approve

The setup code is a base64-encoded JSON payload containing:
- `url` — the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `bootstrapToken` — a short-lived single-device bootstrap token for the initial handshake

Bootstrap token scope rules:
- Primary handed-off `node` token stays `scopes: []`
- Operator token is bounded to the bootstrap allowlist: `operator.approvals`, `operator.read`, `operator.talk.secrets`, `operator.write`
- Bootstrap scope checks are role-prefixed — operator scope entries only satisfy operator requests; non-operator roles must still request scopes under their own role prefix

Treat the setup code like a password while valid.

**CLI management:**

```shellscript
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

If a device retries with different auth details (e.g., different role, scopes, or public key), the previous pending request is superseded and a new `requestId` is issued.

**State files** in `~/.openclaw/devices/`:
- `pending.json` — short-lived pending requests (expire automatically)
- `paired.json` — approved devices and tokens

### Legacy path and durability
The legacy `node.pair.*` API — CLI `openclaw nodes pending|approve|reject|rename` — is a separate gateway-owned pairing store. WebSocket nodes still require device pairing through the modern flow.

The pairing record is the durable source of truth for approved roles: active device tokens stay bounded to the approved role set, and a stray token entry outside those approved roles does not create new access.

## Related Pages
- [[pairing|Pairing]] — the concept generalized across DM and device contexts
- [[openclaw|OpenClaw]] — the gateway implementing pairing
- [[whatsapp-channel|WhatsApp Channel]] — one of the channels defaulting to `pairing` DM policy
