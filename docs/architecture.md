# titaniumClaw Architecture

## Upstream: OpenClaw

titaniumClaw deploys **[OpenClaw](https://github.com/openclaw/openclaw)** — an open-source, self-hosted personal AI assistant.

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

OpenClaw connects to messaging channels (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, Matrix, WebChat) and routes them through a single Gateway control plane.

## Design

titaniumClaw is a **platform-agnostic hardening framework** for OpenClaw deployments. It separates concerns into:

1. **Common layer** (`common/`) — security model, identity isolation, credential lifecycle, install procedures, and audit framework. Shared across all platforms.
2. **Platform layer** (`platforms/<name>/`) — OS-specific bootstrap, service management, firewall, and hardening. One directory per supported hardware target.

```
                    titaniumClaw
                    /          \
              common/        platforms/
             (shared)      /     |     \
                         pi/   macmini/  (future)
                    (piClaw) (miniClaw)
```

## Common Architecture

All platforms share:

- **OpenClaw Gateway** as the deployed service.
- **Loopback-only bind** — gateway never listens on a public interface.
- **Token auth** — every gateway connection requires a strong random token.
- **Identity isolation** — OpenClaw uses service-owned accounts, never personal.
- **Credential lifecycle** — create, store, rotate, revoke for all secrets.
- **Audit framework** — standard pass/fail checks for security verification.

## Platform Architecture

### Raspberry Pi (piClaw)

```
[Your Mac] ---SSH tunnel---> [Pi: 127.0.0.1:18789]
                                  |
                            [systemd unit]
                                  |
                            [openclaw gateway]
                                  |
                            [openclaw user]
                            /var/lib/openclaw
```

- OS: Raspberry Pi OS Lite 64-bit
- Service: systemd with `ProtectHome`, `ProtectSystem`, `NoNewPrivileges`
- Firewall: UFW
- User: dedicated `openclaw` system user

### Mac Mini (miniClaw)

```
[Your Mac/iPhone] ---SSH/Tailscale---> [Mini: 127.0.0.1:18789]
                                            |
                                       [launchd agent]
                                            |
                                       [openclaw gateway]
                                            |
                                       [dedicated user or sandboxed agent]
```

- OS: macOS
- Service: launchd
- Firewall: macOS Application Firewall or pf
- User: dedicated macOS user or sandboxed agent
- Bonus: can run OpenClaw macOS node (voice wake, canvas) on same machine

## Deployment Modes

All platforms support:

- **Local-only** (default, v0): gateway on loopback, access via SSH tunnel.
- **Tailscale** (future): gateway on loopback, Tailscale Serve/Funnel handles exposure. OpenClaw has native Tailscale integration.
- **Reverse proxy** (future): gateway on loopback, nginx/caddy handles TLS termination.

## Security Layers

```
Layer 1: Network         — Firewall (default deny) + loopback bind
Layer 2: Authentication  — SSH key-only + gateway token + DM pairing
Layer 3: Authorization   — Service user with no shell + filesystem isolation
Layer 4: Sandboxing      — systemd/launchd hardening directives
Layer 5: Identity        — Service-owned accounts, no personal data access
Layer 6: Operations      — Health checks, encrypted backups, credential rotation
Layer 7: Audit           — Automated verification of all above layers
```
