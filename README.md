# OpenClaw Infrastructure

A self-hosted, multi-user AI agent platform running on an M2 Mac Mini home server.
Routes Slack commands to isolated per-user agent instances, each backed by n8n workflows
and connected to Claude and Gemini for task execution.

---

## What It Does

Each user gets a private agent accessible via Slack — triggerable by command or mention.
The agent handles tasks like email triage, morning briefings, and workflow automation,
with results posted back to Slack asynchronously.

The platform is designed to be multi-tenant from the ground up: users are isolated at the
container level, provisioned on demand, and never share state or credentials.

---

## Stack

| Layer | Tool |
|---|---|
| Agent framework | OpenClaw |
| Workflow orchestration | n8n (self-hosted) |
| AI models | Claude API (Anthropic), Gemini (Google AI Studio) |
| Routing | Node.js |
| Containerisation | Docker + Docker Compose |
| Networking | Tailscale Funnel |
| Host | Apple Mac Mini M2, macOS |

---

## Key Engineering Decisions

**Per-user container isolation** — each agent instance runs in its own container with
independent data, credentials, and restart lifecycle. One user's broken workflow has no
blast radius.

**HMAC-SHA256 request verification** — every inbound Slack payload is signature-verified
before dispatch, with replay attack mitigation via timestamp validation. The router
handles this at the edge before any application logic runs.

**Tailscale overlay networking** — the host is never exposed to the public internet.
Slack reaches the router via Tailscale Funnel, which handles TLS termination. No open
ports, no reverse proxy complexity.

**Provisioning by script** — new users are onboarded in a single command. No manual
config editing; the provisioner validates inputs, generates the required files, and
prints exact next steps.
