# titaniumClaw

Hardened deployment profiles for running [OpenClaw](https://github.com/openclaw/openclaw) as a secure, always-on personal AI assistant — on any hardware you own.

## Warning

OpenClaw and agentic AI systems are powerful but immature. Security guardrails, sandboxing, and safety mechanisms are not yet robust across the ecosystem. This project attempts to harden the deployment surface, but it cannot guarantee complete protection against all threat vectors — including ones introduced by the AI agent itself.

This warning applies to titaniumClaw's own agent-driven installation process as well. The playbooks that set up these systems are designed to be executed by AI agents. That means an AI agent will have SSH access to your devices and will run commands on them. Understand what that means before proceeding.

**Use common sense. Review what the agent does. Limit network exposure. Do not trust any AI system with unsupervised access to sensitive data or critical infrastructure without understanding the risks.**

## What Is OpenClaw

[OpenClaw](https://openclaw.ai) is an open-source, self-hosted personal AI assistant ([GitHub](https://github.com/openclaw/openclaw), [docs](https://docs.openclaw.ai), MIT license). It runs a Gateway control plane on your own hardware and connects to the messaging channels you already use — WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix, and more. Runtime: Node.js >= 22. Default gateway port: 18789, bound to loopback.

## Approach: Markdown-First

titaniumClaw does not ship shell scripts, Ansible playbooks, or Terraform configs. All setup, hardening, deployment, and operational procedures are written as **detailed Markdown documents**.

- A human can read a document and follow it step by step.
- An AI agent can read the same document and execute it autonomously.
- No proprietary tooling, no vendor lock-in, no runtime dependencies beyond SSH.
- The detail level is calibrated so that smooth, uninvolved, secure setup is possible — by either a person or an agent.

**The instructions are the product.**

## Platforms

titaniumClaw provides hardened deployment profiles for multiple platforms. Common security practices and procedures are shared; platform-specific steps are separated.

| Platform | Codename | Hardware | Status |
|---|---|---|---|
| [Raspberry Pi](platforms/pi/) | **piClaw** | Pi 4/5, ARM64, Pi OS Lite | Active — design complete |
| [Mac Mini](platforms/macmini/) | **miniClaw** | Apple Silicon, macOS | Scaffolded — design in progress |

More platforms can be added by creating a new directory under `platforms/` and referencing the common docs.

## Design Principles

- **CLI-first** — all management is terminal/CLI. No web dashboards.
- **Identity isolation** — OpenClaw gets its own email, Discord, and API accounts. It never touches personal credentials or data. Only chat sessions and operational config are accessible.
- **Agent-driven** — an AI agent reads the docs, SSHes into the target device, executes step by step, verifies each result, and logs progress. A human can do the same.
- **Minimal surface** — lean OS, strict firewall, key-only SSH, unattended security updates.
- **Defense in depth** — multiple overlapping controls: firewall + bind policy + token auth + user isolation + service sandboxing.

## Repository Layout

```
titaniumClaw/
  common/
    identity-isolation.md      # Shared identity policy
    security-model.md          # Threat model and security principles
    credential-lifecycle.md    # Create / store / rotate / revoke
    openclaw-install.md        # Platform-agnostic OpenClaw setup
    audit-framework.md         # Common audit check structure
  platforms/
    pi/                        # piClaw — Raspberry Pi
      README.md
      config/
        openclaw-gateway.service
        .env.example
    macmini/                   # miniClaw — Mac Mini
      README.md
  docs/
    architecture.md            # Umbrella architecture
    roadmap.md                 # Cross-platform milestones
    hardening-checklist.md     # Universal checklist
  README.md
```

## Quick Start

1. Pick your platform: [Raspberry Pi](platforms/pi/) or [Mac Mini](platforms/macmini/).
2. Read the common [security model](common/security-model.md) to understand the threat model.
3. Follow the platform README for hardware-specific setup.
4. Use the common docs for identity isolation, credentials, and auditing.

## Security Model (summary)

- Gateway binds to `127.0.0.1` only. External access via SSH tunnel or Tailscale.
- Strong random gateway token, rotated quarterly.
- Dedicated service user with no login shell and restricted filesystem access.
- Service sandboxing (systemd on Linux, launchd on macOS).
- Firewall: default deny inbound, SSH only.
- SSH: key-only auth, no root login, no password auth.
- Optional: YubiKey FIDO2 for hardware-bound SSH keys.
- All secrets stored outside git with `600` permissions.
- OpenClaw DM pairing enabled (unknown senders must be approved).

Full threat model and controls: [common/security-model.md](common/security-model.md).

## Identity Isolation

OpenClaw operates under its own service identity:

- Dedicated email account (not personal).
- Dedicated Discord bot account (not personal).
- Dedicated API keys for any external services.
- No access to personal home directories, browser data, or credentials.
- The only data OpenClaw sees: its own chat sessions, conversation history, and operational config.

Full policy: [common/identity-isolation.md](common/identity-isolation.md).

## License

MIT
