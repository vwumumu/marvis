---
title: "WhatsApp Channel"
type: entity
sources:
  - "raw/articles/Chat Channels.md"
  - raw/articles/WhatsApp.md
aliases:
  - OpenClaw WhatsApp
  - "@openclaw/whatsapp"
category: tool
tags:
  - openclaw
  - whatsapp
  - chat-channel
  - baileys
created: 2026-04-17
updated: 2026-04-17
---

# WhatsApp Channel

## Overview
The WhatsApp channel is OpenClaw's production-ready integration with WhatsApp Web via the Baileys library. The Gateway owns linked sessions and reconnect loops, and outbound sends require an active listener for the target account (source: [[whatsapp-source|WhatsApp]]).

## Key Details
- **Package:** `@openclaw/whatsapp` (installed on-demand via onboarding or `openclaw channels add --channel whatsapp`) (source: [[whatsapp-source|WhatsApp]])
- **Transport:** WhatsApp Web via [[baileys|Baileys]]; no separate Twilio WhatsApp channel in the built-in registry (source: [[whatsapp-source|WhatsApp]])
- **Status:** Production-ready (source: [[whatsapp-source|WhatsApp]])
- **Pairing:** QR pairing required on initial link (source: [[chat-channels-source|Chat Channels]])
- **Runtime requirement:** Node runtime; Bun is flagged as incompatible for stable gateway operation (source: [[whatsapp-source|WhatsApp]])
- **Recommended deployment:** Dedicated WhatsApp number (separate identity); personal-number fallback supported with self-chat safeguards (source: [[whatsapp-source|WhatsApp]])
- **Credentials path:** `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` with `.bak` backup (source: [[whatsapp-source|WhatsApp]])
- **Multi-account:** Supported via `channels.whatsapp.accounts.<id>` with per-account overrides for policies, media caps, and reaction levels (source: [[whatsapp-source|WhatsApp]])

## Access Control
- **DM policies:** `pairing` (default), `allowlist`, `open` (requires `*` in `allowFrom`), `disabled` (source: [[whatsapp-source|WhatsApp]])
- **Allowlist format:** E.164-style numbers, normalized internally (source: [[whatsapp-source|WhatsApp]])
- **Group policy + allowlists + mention gating** available; duplicate JSON5 keys cause later entries to override earlier (source: [[whatsapp-source|WhatsApp]])

## Message Handling
- **Text chunking:** `textChunkLimit = 4000` default; modes `length` or `newline` (paragraph-preferred) (source: [[whatsapp-source|WhatsApp]])
- **Media types:** image, video, audio (PTT voice-note with `audio/ogg; codecs=opus`), document; GIF via `gifPlayback: true` (source: [[whatsapp-source|WhatsApp]])
- **Media size cap:** `mediaMaxMb` default 50 for both directions; images auto-optimized (source: [[whatsapp-source|WhatsApp]])
- **Read receipts:** Enabled by default; disable via `sendReadReceipts: false`; self-chat turns always skip (source: [[whatsapp-source|WhatsApp]])
- **Reactions:** `reactionLevel` (off/ack/minimal/extensive), default `minimal`; ack reactions via `channels.whatsapp.ackReaction` (source: [[whatsapp-source|WhatsApp]])
- **Group history injection:** up to `historyLimit: 50` unprocessed messages buffered as context when bot is triggered; `0` disables (source: [[whatsapp-source|WhatsApp]])

## Ignored Message Types
Status (`@status`) and broadcast (`@broadcast`) chats are always ignored (source: [[whatsapp-source|WhatsApp]]).

## Appearances in Sources
- [[whatsapp-source|WhatsApp]] — primary channel documentation
- [[chat-channels-source|Chat Channels]] — listed as a built-in channel using Baileys

## Related Entities
- [[openclaw|OpenClaw]] — the gateway that hosts this channel
- [[baileys|Baileys]] — underlying WhatsApp Web library
