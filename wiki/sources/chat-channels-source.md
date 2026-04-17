---
title: "Chat Channels"
type: source
sources:
  - "raw/articles/Chat Channels.md"
source_path: "raw/articles/Chat Channels.md"
tags:
  - messaging-gateway
  - chat-channels
  - openclaw
created: 2026-04-16
updated: 2026-04-16
---

# Chat Channels

## Key Takeaways
- OpenClaw supports 26+ chat channels spanning consumer messaging, enterprise platforms, decentralized protocols, self-hosted solutions, and telephony (source: [[chat-channels-source|Chat Channels]])
- Channels connect via the Gateway; text is universal, while media and reaction support varies by channel (source: [[chat-channels-source|Chat Channels]])
- Channels can run simultaneously — OpenClaw routes per chat across all active channels (source: [[chat-channels-source|Chat Channels]])
- Telegram offers the fastest setup (simple bot token); WhatsApp requires QR pairing and stores more state on disk (source: [[chat-channels-source|Chat Channels]])
- BlueBubbles is the recommended iMessage integration, replacing the deprecated legacy imsg CLI approach (source: [[chat-channels-source|Chat Channels]])
- DM pairing and allowlists are enforced for safety across all channels (source: [[chat-channels-source|Chat Channels]])

## Detailed Notes
The Chat Channels documentation provides a comprehensive listing of all messaging platforms OpenClaw connects to, organized by integration type.

**Built-in channels** include Discord (Bot API + Gateway), Google Chat (HTTP webhook), Signal (signal-cli), Slack (Bolt SDK), Telegram (grammY Bot API), WebChat (WebSocket), and WhatsApp (Baileys with QR pairing).

**Bundled plugin channels** extend the gateway with BlueBubbles (iMessage), Feishu/Lark (WebSocket), IRC, LINE (Messaging API), Matrix, Mattermost (Bot API + WebSocket), Microsoft Teams (Bot Framework), Nextcloud Talk, Nostr (NIP-04 DMs), QQ Bot, Synology Chat (webhooks), Tlon (Urbit-based), Twitch (IRC), Zalo (Bot API), and Zalo Personal (QR login).

**External plugin channels** installed separately include Voice Call (Plivo/Twilio telephony) and WeChat (Tencent iLink Bot via QR login).

The legacy iMessage integration via imsg CLI is deprecated in favor of BlueBubbles. BlueBubbles provides full feature support (edit, unsend, effects, reactions, group management), though edit is currently broken on macOS 26 Tahoe.

Group behavior varies by channel, and model providers are configured separately from channels.

## Related Pages
- [[openclaw|OpenClaw]] — the gateway these channels connect through
- [[ai-agent-gateway|AI Agent Gateway]] — architectural pattern enabling multi-channel support
