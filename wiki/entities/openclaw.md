---
title: "OpenClaw"
type: entity
sources:
  - raw/articles/OpenClaw.md
  - "raw/articles/Chat Channels.md"
  - raw/articles/WhatsApp.md
  - raw/articles/Pairing.md
  - raw/articles/Groups.md
aliases:
  - OpenClaw Gateway
category: tool
tags:
  - ai-agent
  - messaging-gateway
  - self-hosted
  - open-source
  - pairing
  - group-chat
created: 2026-04-16
updated: 2026-04-20
---

# OpenClaw

## Overview
OpenClaw is a self-hosted, open-source gateway that connects messaging apps to AI coding agents. It runs as a single Gateway process on the user's own hardware, bridging channels like Discord, iMessage, Signal, Slack, Telegram, and WhatsApp to an always-available AI assistant (source: [[openclaw-source|OpenClaw]]).

## Key Details
- **Type:** Self-hosted AI agent gateway (source: [[openclaw-source|OpenClaw]])
- **License:** MIT, community-driven (source: [[openclaw-source|OpenClaw]])
- **Runtime:** Node 24 (recommended) or Node 22 LTS (22.14+) (source: [[openclaw-source|OpenClaw]])
- **Default agent:** Bundled Pi binary in RPC mode (source: [[openclaw-source|OpenClaw]])
- **Configuration:** `~/.openclaw/openclaw.json` (source: [[openclaw-source|OpenClaw]])
- **Control UI:** Browser dashboard at `http://127.0.0.1:18789/` for chat, config, and sessions (source: [[openclaw-source|OpenClaw]])
- **Supported channels (26+):** Built-in: Discord, Google Chat, Signal, Slack, Telegram, WebChat, WhatsApp. Bundled plugins: BlueBubbles (iMessage), Feishu/Lark, IRC, LINE, Matrix, Mattermost, Microsoft Teams, Nextcloud Talk, Nostr, QQ Bot, Synology Chat, Tlon, Twitch, Zalo, Zalo Personal. External plugins: Voice Call (Plivo/Twilio), WeChat. Legacy: iMessage via imsg CLI (deprecated) (source: [[chat-channels-source|Chat Channels]])
- **Fastest channel setup:** Telegram (simple bot token); WhatsApp requires QR pairing and stores more state (source: [[chat-channels-source|Chat Channels]])
- **Recommended iMessage integration:** BlueBubbles, replacing deprecated imsg CLI (source: [[chat-channels-source|Chat Channels]])
- **Mobile nodes:** iOS and Android with Canvas, camera, and voice workflows (source: [[openclaw-source|OpenClaw]])
- **Key features:** Multi-channel messaging, multi-agent routing, media support, plugin extensibility, workspace isolation (source: [[openclaw-source|OpenClaw]])
- **Owner-approval gating:** Both DM access and device/node onboarding use the [[pairing|pairing]] flow — 8-char codes approved via `openclaw pairing approve <channel> <CODE>`; devices approved via `openclaw devices approve <requestId>` (source: [[pairing-source|Pairing]])
- **State directories:** `~/.openclaw/credentials/` holds DM pairing state (`<channel>-pairing.json`, `<channel>-allowFrom.json`, plus account-scoped variants); `~/.openclaw/devices/` holds device pairing state (`pending.json`, `paired.json`) (source: [[pairing-source|Pairing]])
- **Uniform group model:** [[group-policy|Group Policy]] applies identically across nine surfaces (Discord, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, Zalo) — three-gate evaluation (`groupPolicy` → allowlists → mention gating) with fail-closed `allowlist` default (source: [[groups-source|Groups]])
- **Session-key isolation:** DMs use the main session; groups always use `agent:<agentId>:<channel>:group:<id>` (rooms use `:channel:<id>`, Telegram forum topics append `:topic:<threadId>`) — enabling single-agent "DMs on host, groups sandboxed" via `sandbox.mode: "non-main"` (source: [[groups-source|Groups]])
- **Documentation:** https://docs.openclaw.ai/ (source: [[openclaw-source|OpenClaw]])

## Appearances in Sources
- [[openclaw-source|OpenClaw]] — primary documentation source
- [[chat-channels-source|Chat Channels]] — detailed channel listing and integration methods
- [[whatsapp-source|WhatsApp]] — per-channel configuration reference exemplifying the gateway's policy model
- [[pairing-source|Pairing]] — DM and device pairing (owner-approval) flows
- [[groups-source|Groups]] — unified group-chat access control across nine channels

## Related Entities
- [[pi-agent|Pi]] — AI coding agent bundled with OpenClaw
- [[whatsapp-channel|WhatsApp Channel]] — production-ready channel using Baileys
- [[baileys|Baileys]] — WhatsApp Web library backing the WhatsApp channel

## Related Concepts
- [[ai-agent-gateway|AI Agent Gateway]] — the architectural pattern OpenClaw implements
- [[pairing|Pairing]] — owner-approval gate for DMs and device onboarding
- [[group-policy|Group Policy]] — unified group-chat access model applied across channels
