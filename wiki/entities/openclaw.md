---
title: "OpenClaw"
type: entity
sources:
  - raw/articles/OpenClaw.md
aliases:
  - OpenClaw Gateway
category: tool
tags:
  - ai-agent
  - messaging-gateway
  - self-hosted
  - open-source
created: 2026-04-16
updated: 2026-04-16
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
- **Supported channels:** Discord, Google Chat, iMessage, Matrix, Microsoft Teams, Signal, Slack, Telegram, WhatsApp, Zalo, WebChat, plus plugin channels (Nostr, Twitch) (source: [[openclaw-source|OpenClaw]])
- **Mobile nodes:** iOS and Android with Canvas, camera, and voice workflows (source: [[openclaw-source|OpenClaw]])
- **Key features:** Multi-channel messaging, multi-agent routing, media support, plugin extensibility, workspace isolation (source: [[openclaw-source|OpenClaw]])
- **Documentation:** https://docs.openclaw.ai/ (source: [[openclaw-source|OpenClaw]])

## Appearances in Sources
- [[openclaw-source|OpenClaw]] — primary documentation source

## Related Entities
- [[pi-agent|Pi]] — AI coding agent bundled with OpenClaw

## Related Concepts
- [[ai-agent-gateway|AI Agent Gateway]] — the architectural pattern OpenClaw implements
