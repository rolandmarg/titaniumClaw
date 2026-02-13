# piClaw Architecture

## Goals

- Run OpenClaw reliably on Raspberry Pi hardware.
- Keep attack surface small by default.
- Make operations reproducible and observable.

## Core Components

1. Host OS (Raspberry Pi OS 64-bit)
   - base hardening
   - package lifecycle and updates
2. OpenClaw Runtime
   - Node.js 22 runtime
   - OpenClaw gateway process
3. Service Management
   - systemd unit with restart policy
   - least-privilege service user
4. Security Controls
   - token auth
   - loopback bind by default
   - nftables/ufw rules
5. Ops Layer
   - structured logs via journald
   - health checks and restart validation
   - backup + restore workflow

## Deployment Modes

- Local-only mode (default)
  - Gateway binds to loopback.
  - Access through SSH tunnel.
- Reverse-proxy mode (optional)
  - Gateway still loopback.
  - Reverse proxy handles TLS/auth boundaries.
  - Trusted proxy list must be explicit.

## Security Baseline

- Dedicated system user for runtime.
- Strong random `OPENCLAW_GATEWAY_TOKEN`.
- Secrets stored outside git, file permissions `600`.
- SSH key-only login and password auth disabled.
- Unneeded services disabled.

## Assumptions

- Single-node deployment for v0.
- Personal usage profile (not multi-tenant).
