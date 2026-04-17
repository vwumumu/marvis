---
title: "OpenClaw"
type: source
sources:
  - raw/articles/OpenClaw.md
source_path: raw/articles/OpenClaw.md
tags:
  - ai-agent
  - messaging-gateway
  - self-hosted
  - open-source
created: 2026-04-16
updated: 2026-04-16
---

# OpenClaw

## Key Takeaways
- OpenClaw is a self-hosted, open-source gateway that connects messaging apps (Discord, iMessage, Signal, Slack, Telegram, WhatsApp, and more) to AI coding agents like Pi (source: [[openclaw-source|OpenClaw]])
- A single Gateway process bridges multiple chat channels simultaneously, including built-in channels, bundled plugins, WebChat, and mobile nodes (source: [[openclaw-source|OpenClaw]])
- Built for coding agents with tool use, sessions, memory, and multi-agent routing with isolated per-agent/per-sender sessions (source: [[openclaw-source|OpenClaw]])
- Supports mobile nodes on iOS and Android with Canvas, camera, and voice-enabled workflows (source: [[openclaw-source|OpenClaw]])
- Requires Node 24 (recommended) or Node 22 LTS (22.14+), an API key, and minimal setup time (source: [[openclaw-source|OpenClaw]])
- MIT licensed and community-driven (source: [[openclaw-source|OpenClaw]])

## Detailed Notes
OpenClaw is a self-hosted gateway designed for developers and power users who want a personal AI assistant accessible from any messaging platform without sacrificing data control. The Gateway process is the single source of truth for sessions, routing, and channel connections.

The system supports a wide range of channels: Discord, Google Chat, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, Zalo, and WebChat. Additional channels (Matrix, Nostr, Twitch, Zalo) are available through bundled plugin channels.

By default, OpenClaw uses a bundled Pi binary in RPC mode with per-sender sessions, requiring no configuration. Optional configuration lives at `~/.openclaw/openclaw.json` and supports allowlists, mention patterns for group chats, and provider settings.

Key capabilities include multi-channel messaging, plugin-based channel extensibility, multi-agent routing with workspace isolation, media support (images, audio, documents), a browser-based Web Control UI at `http://127.0.0.1:18789/`, and mobile nodes for iOS and Android with Canvas, camera, and device action support. Remote access is possible via SSH, Tailscale, or web surfaces.

## Related Pages
- [[openclaw|OpenClaw]] — the gateway tool itself
- [[pi-agent|Pi]] — the AI coding agent bundled with OpenClaw
- [[ai-agent-gateway|AI Agent Gateway]] — the architectural pattern OpenClaw implements
