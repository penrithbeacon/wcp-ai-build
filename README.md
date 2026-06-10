# WCP AI Build

> Build any WCP artefact — guided by AI, from first question to running container.

**[Widget Context Protocol (WCP)](https://widgetcontextprotocol.com)** defines what WCP
artefacts must do. This repository defines how to build them — with AI as the implementer
and you as the architect.

---

## Quick Start

Give your AI a single instruction:

> _"Read https://github.com/penrithbeacon/wcp-ai-build/blob/main/AI-SKILL.md
> and help me build a WCP artefact."_

Your AI will orient you in the WCP ecosystem, ask what you want to build, and hand you off
to the appropriate specialist skill.

---

## What You Can Build

| What | Give your AI this URL |
|------|-----------------------|
| **Anything** — not sure yet, start here | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build/blob/main/AI-SKILL.md) |
| **A widget** — containerised UI component for the WCP dashboard | [wcp-ai-build-widget/AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-widget/blob/main/AI-SKILL.md) |
| **An agent** — native host process that serves data to widgets | [wcp-ai-build-agent/AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-agent/blob/main/AI-SKILL.md) |
| **A macOS agent** — native macOS background service | [wcp-ai-build-agent-mac/AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-agent-mac/blob/main/AI-SKILL.md) |

---

## Builder's Guides

The [`guides/`](guides/) folder contains real-world build examples — distilled conversations
between developer/architect and AI coding agent. Each guide shows a complete build journey
from first question to running artefact.

If you are new to WCP, read a guide before starting your first build. The examples walk in
from the shallow end: intent expressed in plain language, AI questions and decisions, and
the final running code.

---

## After You've Built

Once your artefact is built and running, use **WCP AI Release** to document, audit,
and publish to Docker Hub:

> _"Read https://github.com/penrithbeacon/wcp-ai-release/blob/main/AI-SKILL.md
> and release my artefact."_

---

## Links

- [Widget Context Protocol](https://widgetcontextprotocol.com)
- [WCP Developer Guide](https://dev.widgetcontextprotocol.com)
- [WCP AI Release](https://github.com/penrithbeacon/wcp-ai-release) — documentation, auditing, deployment
- [Penrith Beacon](https://penrithbeacon.com)
- [Docker Hub — penrithbeacon](https://hub.docker.com/u/penrithbeacon)
