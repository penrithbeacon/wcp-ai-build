# WCP AI Build — AI Skill

> **Version 1.0.0** | For consumption by any AI engine assisting with WCP artefact creation.
> Source of truth: [github.com/penrithbeacon/wcp-ai-build](https://github.com/penrithbeacon/wcp-ai-build)

---

## 1. What This Repository Is

This repository is the **entry point** for building any WCP artefact with AI assistance.

The [Widget Context Protocol (WCP)](https://widgetcontextprotocol.com) ecosystem has two
kinds of artefact:

| Artefact | What it is |
|----------|-----------|
| **Widget** | A containerised web application (Docker) that exposes a UI and an HTTP API. Runs anywhere Docker runs. Registered with a WCP host dashboard. |
| **Agent** | A native process running directly on the host machine. Exposes a local HTTP API that widget containers reach via `host.docker.internal`. Provides access to host data and capabilities. |

This repo does **not** perform builds. It orients, defines the universal patterns used
across all build repos, and routes to the appropriate specialist skill.

---

## 2. Technology Negotiation Pattern

**This is the canonical definition.** All `wcp-ai-build-*` sub-repos reference this section
rather than repeating it.

Every build involves at least one technology decision (language, framework, packaging tool).
Apply this pattern whenever a technology choice must be made:

### Step 1 — Ask first

Give the developer the opportunity to state their preferred technology before offering options.

Example: _"Do you have a preferred language or framework for this widget, or would you like
me to suggest options?"_

### Step 2 — If no preference is stated

Present a curated list of 3–4 industry-standard options. For each option provide:
- Name and one-line description
- Typical use case / who typically uses it
- Key trade-off in one sentence

Research current options at the time of the build — the list should reflect what is in
active, widespread production use at the time of reading.

### Step 3 — State the AI default explicitly

One option is labelled **AI default**. This is the choice the AI will make if the developer
defers the decision entirely. The AI default must meet all of these criteria:

- In active, widespread production use at major organisations globally
- Well-documented, with a large and active community
- Conservatively modern — not leading-edge, not legacy
- The choice a senior engineer at a blue-chip organisation would make without internal controversy

The default is **not a recommendation** — it is simply the AI's fallback. A developer who
names a specific technology overrides it immediately and without friction.

### Step 4 — Proceed on deferral

If the developer says "your choice", "whatever you think", or does not respond with a
technology preference, immediately select the AI default and proceed. Do not ask again.

---

## 3. Skill: Determine What to Build

Use this skill when a developer asks you to help them build a WCP artefact.

### Step 1 — Orient the developer

Ask the following questions and **record every answer**. These answers must be carried
forward verbatim when routing to the specialist skill — do not rely on the specialist
skill to re-ask them.

1. Are they working with an existing WCP host dashboard (e.g. Penrith Beacon Design Studio),
   or starting from scratch?
2. Do they already have other widgets or agents running?
3. If yes to (2): which ports are those widgets and agents using? List them.
   Do not assume any ports are free or occupied — ask explicitly.
4. What port is the WCP Bonjour discovery service running on?
   (The default is 3737, but it may have been configured differently.
   If Bonjour is not yet running, note this — it will affect service discovery for the new artefact.)

### Step 2 — Ask what they want to build

_"What would you like to build?"_

The developer may answer in abstract terms (features, benefits, intended use) or in
technical terms (widget, agent, specific technology). Both are valid — abstract answers
are refined during the design phase in the specialist skill.

### Step 3 — Route to the appropriate specialist skill

| Developer says | Action |
|----------------|--------|
| A widget / dashboard component / UI panel / something in a card | Read [wcp-ai-build-widget AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-widget/blob/main/AI-SKILL.md) and follow it |
| An agent / host service / background process / something that reads local data | Read [wcp-ai-build-agent AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-agent/blob/main/AI-SKILL.md) and follow it |
| Both a widget and an agent | Complete the agent skill first (it defines the data source), then the widget skill (it consumes it) |
| Unsure | Ask clarifying questions about the desired outcome — what will the user see or do? Map to widget if it has a UI, agent if it only serves data |

**Carry the Step 1 context answers forward** when handing off to the specialist skill.
Do not attempt to perform the build from this document.

**Note on host applications:** If the developer's answer suggests they want to build a WCP
*host* (a dashboard application) rather than a widget or agent, clarify — host development
is outside the scope of this namespace.

---

## 4. WCP Ecosystem Context

Reference this section when orienting a developer who is new to WCP.

### Protocol version

WCP 2.1.0 — specification at [widgetcontextprotocol.com](https://widgetcontextprotocol.com)

### Widgets

- Run inside Docker containers — `docker run` or `docker compose up`
- Expose HTTP endpoints; the WCP protocol defines which are mandatory
- Serve HTML directly into iframe cards in the host dashboard
- Communicate with the dashboard via postMessage (theme, context, orchestration state)
- Registered with the host by pointing to the container's port
- Can be published to Docker Hub and pulled by any developer worldwide
- Build standard: [WIDGET-BUILD-SPEC.md](https://github.com/penrithbeacon/wcp-ai-automation/blob/main/standards/WIDGET-BUILD-SPEC.md)

### Agents

- Run natively on the host machine — NOT in Docker
- Expose a local HTTP API on a loopback address (`127.0.0.1`)
- Widget containers reach them via the Docker bridge: `host.docker.internal:<port>`
- Have access to host resources: file system, installed tools, running processes, OS APIs
- Must auto-start at user login for production use
  (launchd on macOS, systemd on Linux, Task Scheduler on Windows)
- Have no UI of their own — they serve data; widgets present it

### How they relate

```
[WCP Host Dashboard]
        │  iframe cards
        ▼
[Widget Container :port]  ──── host.docker.internal ────▶  [Agent :port on host]
        │                                                          │
   HTTP + WCP API                                          host data & APIs
```

### Reference implementations

- 9 WCP widgets: [github.com/penrithbeacon](https://github.com/penrithbeacon) (wcp-widget-*)
- CLAUD agent (macOS, Apple Silicon): `widgets/claude/agents/mac-arm64/` in
  [wcp-widget-claude](https://github.com/penrithbeacon/wcp-widget-claude)

---

## 5. Builder's Guides

The [`guides/`](guides/) folder in this repository contains real-world build examples.

Each guide records a complete build journey: the architect's intent expressed in plain
language, the AI's questions and decisions, and the final artefact. Conversations are
distilled and corrected for clarity but preserve the natural progression from vague idea
to running code.

If you are reading this for the first time, check the [guides index](guides/README.md) for
an example relevant to what you want to build. The guides walk in from the shallow end —
each one is readable by developers at any level.

---

## 6. Cross-References

| Repository | Purpose | AI Skill |
|-----------|---------|---------|
| [wcp-ai-build](https://github.com/penrithbeacon/wcp-ai-build) | Entry point — this repo | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build/blob/main/AI-SKILL.md) |
| [wcp-ai-build-widget](https://github.com/penrithbeacon/wcp-ai-build-widget) | Build a WCP widget | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-widget/blob/main/AI-SKILL.md) |
| [wcp-ai-build-agent](https://github.com/penrithbeacon/wcp-ai-build-agent) | Build a WCP agent (platform-agnostic design) | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-agent/blob/main/AI-SKILL.md) |
| [wcp-ai-build-agent-mac](https://github.com/penrithbeacon/wcp-ai-build-agent-mac) | Build a macOS native agent | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-build-agent-mac/blob/main/AI-SKILL.md) |
| [wcp-ai-automation](https://github.com/penrithbeacon/wcp-ai-automation) | Document, audit, and deploy a widget | [AI-SKILL.md](https://github.com/penrithbeacon/wcp-ai-automation/blob/main/AI-SKILL.md) |
