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

**Question 1:** Are they working with an existing WCP host dashboard (e.g. Penrith Beacon
Design Studio), or starting from scratch?

**Question 2:** What port is the WCP Bonjour discovery service running on?
(The default is 3737, but it may have been configured differently.)

#### If the developer does not know the Bonjour port

Apply the following decision tree:

**Case A — Developer is using Penrith Beacon (or has the WCP Bonjour Proxy installed)**

The WCP Bonjour Proxy is a macOS menu bar agent (`wcp-bonjour-proxy`) that:
- Runs as an icon near the clock in the top-right of the screen
- On startup, automatically discovers any running Bonjour service across all ports
- If no Bonjour service is found, pulls and starts `penrithbeacon/wcp-widget-bonjour`
  automatically from Docker Hub — the user does not need to install Bonjour separately
- Displays both the proxy port and the Bonjour service port in its menu

Tell the developer:

> _"The WCP Bonjour Proxy should be running as a menu bar icon near the clock at
> the top right of your screen. Tap that icon — it will show both the proxy port
> and the Bonjour service port. If Bonjour was not already running, the proxy will
> have started it automatically. Give me those two port numbers and we can proceed."_

Once the developer confirms the Bonjour port (typically 3737), record it and proceed.

**Case B — Developer is using a different WCP host dashboard**

Ask the developer to describe what they know about their host dashboard and its
configuration. Use that information to help identify the Bonjour port — for example,
by checking configuration files, environment variables, or docker-compose.yml for the
host, or by querying `docker ps` to identify a running Bonjour container.

If no Bonjour service is running at all, recommend the developer install
`wcp-bonjour-proxy` from `https://github.com/penrithbeacon/wcp-bonjour-proxy` — it
will bootstrap the Bonjour service automatically on first launch.

**Case C — Port cannot be determined**

If the developer cannot confirm the Bonjour port after working through Cases A or B:

> **Do not proceed.** The Bonjour port is required before this skill can continue.
> Recommend the developer install `wcp-bonjour-proxy`, which will start Bonjour
> automatically. Once they have confirmed the Bonjour port from the menu bar, resume.

Proceeding without a confirmed Bonjour port would result in incorrect port assignments
for the new artefact. There is no safe fallback.

---

> ⛔ **SECURITY CONSTRAINT — Bonjour must be on the local machine, loopback-bound**
>
> The Bonjour service MUST be running on `127.0.0.1` (the local machine). This is a hard
> security requirement, not a preference. The WCP Bonjour service does not currently
> implement network-level security. Querying it over a local network would expose widget
> and agent registrations to other machines on that network.
>
> **This skill will not query a Bonjour service at any address other than `127.0.0.1`.**
>
> **Technical enforcement — docker-compose.yml port binding:**
> The Bonjour container's `docker-compose.yml` MUST bind the port as:
> ```yaml
> ports:
>   - "127.0.0.1:3737:3737"
> ```
> The form `"3737:3737"` (without the `127.0.0.1:` prefix) binds to all interfaces
> (`0.0.0.0`) and is **non-compliant**, regardless of intent or any other configuration.
> This is the technical enforcement of the loopback-only requirement. The AI must flag
> this as an error if it encounters a Bonjour `docker-compose.yml` that uses the
> unbound form.
>
> If the developer states that the Bonjour service is on a network address:
> 1. Refuse to query it and state this constraint clearly.
> 2. If the developer asserts that security has been built into the Bonjour service:
>    - Fetch the Docker Hub documentation for `penrithbeacon/wcp-widget-bonjour`
>      at `https://hub.docker.com/r/penrithbeacon/wcp-widget-bonjour` and read it.
>    - If the documentation **explicitly states** that network security is implemented
>      and safe for local network deployment — proceed.
>    - If the documentation does **not** explicitly state this, or the page does not
>      yet exist — refuse.
> 3. If the developer wishes to proceed regardless, they must clone these skill files
>    locally and edit them to suit their own requirements. This skill will not deviate.
>
> *This constraint will be updated when the Bonjour service publishes verified network
> security support in its Docker Hub documentation.*

---

If Bonjour is not yet running at all, note this and proceed to Question 3 — the developer
will need to list occupied ports manually.

**Question 3:** Ask permission to query the Bonjour service:

_"May I query the Bonjour discovery service at `http://127.0.0.1:{port}/` to find
which ports are already occupied by registered widgets and agents?"_

- **Yes** — query `http://127.0.0.1:{port}/` and extract the list of registered widget
  and agent ports. Record these as the occupied port set. Do not ask the developer to
  list ports manually.
- **No** — ask the developer to list the ports of any currently running widgets and agents.
  Record these as the occupied port set.
- **Bonjour not yet running** — ask the developer to list any occupied ports manually.

**Question 4: GitHub account**

_"Do you have a GitHub account?"_

**Why a GitHub account is needed — explain this if the developer asks or does not have one:**

> A widget lives in two places once it is built. The first is a **GitHub repository** —
> this is where the source code lives, where changes are tracked, and where other
> developers can find and contribute to it. The second is a **Docker Hub image** — this
> is the distributable, runnable artefact that any WCP host can pull and launch.
>
> Here is the full lifecycle from code to running widget on someone else's machine:
>
> 1. You write the widget's source code and push it to a GitHub repository.
> 2. A Docker image is built from that source — a static, versioned snapshot of
>    everything the widget needs to run (code, dependencies, runtime).
> 3. That image is pushed to Docker Hub, where it gets a permanent address called an
>    OCI reference (e.g. `yourname/wcp-widget-example:1.0.0`).
> 4. When you export an orchestration containing your widget, that OCI reference is
>    embedded in the `.WCPA` file.
> 5. Anyone who imports that `.WCPA` file into their WCP host — whether Penrith Beacon
>    Design Studio or any other WCP-compliant host — has the orchestration file pull
>    the image automatically from Docker Hub via the Bonjour service and launch it as
>    a running container.
> 6. A **container** is the live, running instance of the image — it has its own network
>    port, its own isolated filesystem, and its own lifecycle. The host dashboard loads
>    the widget's UI from that container into an iframe card.
>
> Without GitHub: no source control, no collaboration, no open-source presence.
> Without Docker Hub: no distributable image, no automatic pull on import, no WCP
> distribution lifecycle.

- **If the developer has a GitHub account:** ask for their GitHub username and a
  Personal Access Token (PAT) with **`repo` scope** (needed to create the repository,
  push code, and later push documentation).
- **If they do not have an account:** explain the lifecycle above, then walk them through
  creating one at [github.com](https://github.com). Do not proceed with PAT collection
  until the account exists.

**Question 5: Docker Hub account**

_"Do you have a Docker Hub account?"_

*(Docker Hub is required for widget distribution — see the lifecycle explanation above.
Agents do not require Docker Hub; this question is specifically relevant to widget builds.)*

- **If the developer has a Docker Hub account:** ask for their Docker Hub username and
  an **Access Token** with Read, Write, and Delete permissions on repositories (needed
  to push the image and, later, for automated audit and release workflows).
- **If they do not have an account:** direct them to [hub.docker.com](https://hub.docker.com)
  to create a free account. Note that the free tier supports unlimited public repositories,
  which is sufficient for distributable WCP widgets.

**Credentials storage**

Once both tokens are collected, ask the developer where to store them:

> _"I need to save these credentials securely so they can be used throughout the build
> and deployment pipeline. I recommend storing them at `~/wcp-credentials/credentials.md`
> — a folder outside any source repository. Would you like to use that location, or
> specify a different path?"_

Rules:
- **Do not write the credentials file until the developer confirms the path.**
- The default path `~/wcp-credentials/credentials.md` places the file outside any
  project directory and outside any git repository.
- If the developer chooses a path inside a git repository, add the filename to that
  repository's `.gitignore` immediately, before writing the file.
- If the chosen path does not exist, create it with `mkdir -p`.

Credentials file format:

```markdown
# WCP Build Credentials
# ⛔ DO NOT COMMIT THIS FILE TO SOURCE CONTROL
# Store this file outside all git repositories, or ensure it is in .gitignore.

## GitHub
GITHUB_USERNAME=
GITHUB_PAT=

## Docker Hub
DOCKERHUB_USERNAME=
DOCKERHUB_TOKEN=
```

Record the confirmed credentials file path — it must be carried forward to every
subsequent pipeline stage.

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
| Both a widget and an agent | Run both pipelines **in parallel within the same conversation**. Both specialist skills proceed simultaneously, converging at integration points (port assignment, data contract, `host.docker.internal` reference). See parallel pipeline notes in each specialist skill. |
| Unsure | Ask clarifying questions about the desired outcome — what will the user see or do? Map to widget if it has a UI, agent if it only serves data |

**Before routing, announce your decision** — tell the developer which specialist skill you
are moving to and why, so they can correct you if the routing is wrong.

**Carry all of the following forward** when handing off to the specialist skill:
- Step 1 context: host dashboard, Bonjour port, occupied port set
- GitHub username and PAT
- Docker Hub username and access token
- Credentials file path
- The developer's full Step 2 answer — their complete description of what they want to
  build, verbatim. Do not summarise or reduce it. The specialist skill will use this to
  pre-populate answers to its design questions and skip anything already covered.

**Do not ask any design questions from this document.** Design questions are the exclusive
responsibility of the specialist skill. The moment you identify the artefact type, route —
do not begin designing here.

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
