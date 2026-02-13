# piClaw Architecture

## Upstream: OpenClaw

piClaw deploys **[OpenClaw](https://github.com/openclaw/openclaw)** — an open-source, self-hosted personal AI assistant.

| Field | Value |
|---|---|
| Name | OpenClaw |
| Description | Your own personal AI assistant. Any OS. Any Platform. |
| GitHub | https://github.com/openclaw/openclaw |
| Website | https://openclaw.ai |
| Docs | https://docs.openclaw.ai |
| License | MIT |
| Runtime | Node.js >= 22 |
| Default gateway port | 18789 |
| Default gateway bind | loopback (`ws://127.0.0.1:18789`) |
| Config path | `~/.openclaw/openclaw.json` |
| Workspace path | `~/.openclaw/workspace` |

OpenClaw connects to messaging channels (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix, WebChat) and routes them through a single Gateway control plane. The Gateway is the component piClaw deploys and secures on a Raspberry Pi.

## Goals

- Run the OpenClaw Gateway reliably on Raspberry Pi hardware.
- Keep attack surface small by default.
- Make operations reproducible and observable.

## Core Components

1. **Host OS** (Raspberry Pi OS Lite 64-bit)
   - Base hardening, package lifecycle, unattended security updates.
2. **OpenClaw Runtime**
   - Node.js 22 runtime.
   - OpenClaw Gateway process (`openclaw gateway --port 18789 --verbose`).
   - Config at `/var/lib/openclaw/.openclaw/openclaw.json`.
   - Workspace at `/var/lib/openclaw/.openclaw/workspace`.
3. **Service Management**
   - systemd unit with restart policy.
   - Least-privilege `openclaw` service user.
4. **Security Controls**
   - Gateway token auth (env var `OPENCLAW_GATEWAY_TOKEN`).
   - Loopback bind by default.
   - UFW firewall rules.
   - SSH key-only access.
   - DM pairing (OpenClaw's built-in sender verification).
5. **Identity Isolation**
   - Dedicated service email, Discord bot, and API accounts.
   - No personal credentials or data accessible to OpenClaw.
   - Filesystem sandboxing via systemd and Linux user permissions.
6. **Ops Layer**
   - Structured logs via journald.
   - Health checks (`openclaw doctor` + custom timer).
   - Backup + restore workflow.

## Deployment Modes

- **Local-only mode** (default, v0)
  - Gateway binds to loopback.
  - Access through SSH tunnel.
- **Tailscale mode** (future, optional)
  - Gateway stays loopback.
  - Tailscale Serve (tailnet-only) or Funnel (public) handles exposure.
  - OpenClaw has native Tailscale integration (`gateway.tailscale.mode`).
- **Reverse-proxy mode** (future, optional)
  - Gateway still loopback.
  - Reverse proxy handles TLS/auth boundaries.
  - Trusted proxy list must be explicit.

## Security Baseline

- Dedicated system user for runtime.
- Strong random gateway token, rotated on schedule.
- Secrets stored outside git, file permissions `600`.
- SSH key-only login, password auth disabled, root login disabled.
- Unneeded services disabled.
- UFW firewall with default deny inbound.
- OpenClaw DM pairing enabled (unknown senders must be approved).
- Optional YubiKey FIDO2 for SSH (M3b).

## Assumptions

- Single-node deployment for v0.
- Personal usage profile (not multi-tenant).
