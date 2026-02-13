# piClaw

Hardened Raspberry Pi profile for running [OpenClaw](https://github.com/openclaw/openclaw) as a secure, always-on personal AI assistant node.

## What Is OpenClaw

[OpenClaw](https://openclaw.ai) is an open-source, self-hosted personal AI assistant ([GitHub](https://github.com/openclaw/openclaw), [docs](https://docs.openclaw.ai), MIT license). It runs a Gateway control plane on your own hardware and connects to the messaging channels you already use — WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix, and more. Runtime: Node.js >= 22. Default gateway port: 18789, bound to loopback.

## Warning

OpenClaw and agentic AI systems are powerful but immature. Security guardrails, sandboxing, and safety mechanisms are not yet robust across the ecosystem. This project attempts to harden the deployment surface, but it cannot guarantee complete protection against all threat vectors — including ones introduced by the AI agent itself.

This warning applies to piClaw's own agent-driven installation process as well. The playbooks that set up this system are designed to be executed by AI agents. That means an AI agent will have SSH access to your Pi and will run commands on it. Understand what that means before proceeding.

**Use common sense. Review what the agent does. Limit network exposure. Do not trust any AI system with unsupervised access to sensitive data or critical infrastructure without understanding the risks.**

## Approach: Markdown-First

piClaw does not ship shell scripts, Ansible playbooks, or Terraform configs. Instead, all setup, hardening, deployment, and operational procedures are written as **detailed Markdown playbooks**.

Why?

- A human can read a playbook and follow it step by step.
- An AI agent can read the same playbook and execute it autonomously.
- No proprietary tooling, no vendor lock-in, no runtime dependencies beyond SSH.
- The detail level is calibrated so that smooth, uninvolved, secure setup is possible — by either a person or an agent.

The instructions are the product. Everything in this repo is supporting material.

## What piClaw Does

- Turns a Raspberry Pi into a hardened, single-purpose OpenClaw host.
- Binds the gateway to loopback only — access via SSH tunnel.
- Runs OpenClaw under a sandboxed service identity with zero access to personal data.
- Provides repeatable setup, hardening, backup, and recovery — all from Markdown.

## Design Principles

- **CLI-first** — all management is terminal/CLI. No web dashboards.
- **Identity isolation** — OpenClaw gets its own email, Discord, and API accounts. It never touches personal credentials or data. Only chat sessions and operational config are accessible.
- **Agent-driven** — an AI agent (e.g. opencode) reads the playbooks, SSHes into the Pi, executes step by step, verifies each result, and logs progress. A human can do the same.
- **Minimal surface** — lean OS (Pi OS Lite 64-bit, no desktop), strict firewall, SSH key-only, unattended security updates.
- **Single-node** — one Pi, one OpenClaw instance. Simple.

## Repository Layout

```
piClaw/
  config/
    openclaw-gateway.service   # systemd unit template
  docs/
    architecture.md            # system design and deployment modes
    hardening-checklist.md     # security verification checklist
    roadmap.md                 # milestone plan
  .env.example                 # environment variable reference
  .gitignore
  README.md
```

Playbooks (the actual installation/operation instructions) are maintained separately and are not included in this repo. They can be found in the project's companion documentation system.

## Milestones

| Milestone | Description |
|---|---|
| M0 - Preflight | Flash Pi OS Lite, enable SSH, verify connectivity |
| M1 - Bootstrap | Install OS packages, Node.js 22, OpenClaw, create service user |
| M2 - Deploy | Start systemd service, smoke test gateway, verify logging |
| M3 - Harden | Firewall, SSH lockdown, identity isolation, audit pass |
| M3b - YubiKey | Optional FIDO2/U2F hardware key for SSH (physical presence required) |
| M4 - Ops | Health checks, automated backups, restore testing, upgrade paths |

## Quick Start

1. Flash a Raspberry Pi with **Pi OS Lite 64-bit** (headless, SSH + key pre-enabled).
2. Boot the Pi and verify SSH access: `ssh pi@piclaw.local`
3. Follow the bootstrap playbook (M1) — or point an AI agent at it.
4. Continue through deploy (M2), harden (M3), and ops (M4) playbooks.

The goal: go from a fresh SD card to a fully hardened, operational OpenClaw node.

## Security Model

- Gateway binds to `127.0.0.1` only. External access via SSH tunnel.
- Strong random gateway token, rotated on schedule.
- Dedicated `openclaw` system user with no login shell and restricted filesystem access.
- systemd sandboxing: `ProtectHome`, `ProtectSystem`, `NoNewPrivileges`, `PrivateTmp`.
- UFW firewall: default deny inbound, SSH only.
- SSH: key-only auth, no root login, no password auth.
- Optional: YubiKey FIDO2 for hardware-bound SSH keys.
- All secrets stored outside git with `600` permissions.

## Identity Isolation

OpenClaw operates under its own service identity:

- Dedicated email account (not personal).
- Dedicated Discord bot account (not personal).
- Dedicated API keys for any external services.
- No access to personal home directories, browser data, or credentials.
- The only data OpenClaw sees: its own chat sessions, conversation history, and operational config.

## Status

Design and playbook scaffolding complete. Implementation follows milestone-by-milestone.

## License

MIT
