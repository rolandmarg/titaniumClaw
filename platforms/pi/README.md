# piClaw — Raspberry Pi Platform

Raspberry Pi-specific hardening and deployment profile for OpenClaw.

## Overview

piClaw turns a Raspberry Pi into a hardened, single-purpose OpenClaw host with:

- Raspberry Pi OS Lite 64-bit (no desktop, CLI-only)
- systemd service management with restart policy
- UFW firewall with default deny inbound
- SSH key-only access, no root login
- Dedicated `openclaw` system user with filesystem isolation
- systemd sandboxing (`ProtectHome`, `ProtectSystem`, `NoNewPrivileges`, `PrivateTmp`)

## Hardware Requirements

- Raspberry Pi 4 or Pi 5 with 2GB+ RAM
- MicroSD card (32GB+ recommended) or USB SSD
- 5V/3A power supply (5V/5A for Pi 5)
- Ethernet or WiFi connectivity

## Platform-Specific Details

### Service Management

Uses systemd. Service unit template: [config/openclaw-gateway.service](config/openclaw-gateway.service).

### Firewall

UFW (Uncomplicated Firewall). Default deny inbound, allow SSH only.

### User Isolation

Dedicated `openclaw` system user created via `useradd --system`. No login shell. Home at `/var/lib/openclaw`.

### Credential Storage

- `/etc/openclaw/openclaw.env` — gateway config (token, bind, port)
- `/etc/openclaw/identity.env` — service identity credentials
- Both `600` permissions, owned by `openclaw:openclaw`

### Known Risks

- **SD card failure**: SD cards degrade over time. Consider USB SSD for production use. Implement off-device backups.
- **Limited resources**: Pi 4 (2GB) may struggle under heavy load. Monitor memory usage.
- **ARM64**: most Node.js packages work on ARM64, but verify any native dependencies.

## Common References

This platform extends the shared titanium documentation:

- [Identity Isolation](../../common/identity-isolation.md)
- [Security Model](../../common/security-model.md)
- [Credential Lifecycle](../../common/credential-lifecycle.md)
- [OpenClaw Install](../../common/openclaw-install.md)
- [Audit Framework](../../common/audit-framework.md)

## Milestones

| Milestone | Description |
|---|---|
| M0 - Preflight | Flash Pi OS Lite, enable SSH, verify connectivity |
| M1 - Bootstrap | Install packages, Node.js 22, OpenClaw, create service user |
| M2 - Deploy | Start systemd service, smoke test, verify logging |
| M3 - Harden | UFW, SSH lockdown, identity isolation, audit pass |
| M3b - YubiKey | Optional FIDO2 hardware key for SSH |
| M4 - Ops | Health checks, backups, restore testing, upgrades |
