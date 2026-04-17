---
title: "Baileys"
type: entity
sources:
  - "raw/articles/Chat Channels.md"
  - raw/articles/WhatsApp.md
category: tool
tags:
  - whatsapp
  - library
  - chat-integration
created: 2026-04-17
updated: 2026-04-17
---

# Baileys

## Overview
Baileys is the WhatsApp Web protocol library that OpenClaw's WhatsApp channel uses as its transport. It enables the Gateway to own a WhatsApp socket and manage reconnect loops without relying on the official WhatsApp Business API (source: [[whatsapp-source|WhatsApp]]).

## Key Details
- **Role:** Underlying library for OpenClaw's WhatsApp channel (source: [[chat-channels-source|Chat Channels]])
- **Protocol:** WhatsApp Web (source: [[whatsapp-source|WhatsApp]])
- **Pairing:** QR-based on initial link (source: [[chat-channels-source|Chat Channels]])
- **Scope:** The WhatsApp Web-based channel is the only messaging platform channel in current OpenClaw channel architecture — there is no separate Twilio WhatsApp messaging channel in the built-in registry (source: [[whatsapp-source|WhatsApp]])

## Appearances in Sources
- [[whatsapp-source|WhatsApp]] — described as the transport for OpenClaw's WhatsApp channel
- [[chat-channels-source|Chat Channels]] — listed as the WhatsApp integration library

## Related Entities
- [[whatsapp-channel|WhatsApp Channel]] — OpenClaw channel built on Baileys
- [[openclaw|OpenClaw]] — gateway hosting the WhatsApp channel
