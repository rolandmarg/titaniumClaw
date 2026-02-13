# miniClaw — Mac Mini Platform

Mac Mini-specific hardening and deployment profile for OpenClaw.

## Overview

miniClaw turns a Mac Mini into a hardened, always-on OpenClaw host with:

- macOS with minimal attack surface
- launchd service management with keep-alive
- macOS firewall or pf for network control
- Remote Login (SSH) restricted to key-only auth
- Dedicated service user or sandboxed launchd agent
- FileVault full-disk encryption

## Hardware

- Mac Mini (Apple Silicon recommended)
- Wired ethernet preferred for always-on reliability
- SSD (built-in, no SD card failure risk)

## Platform-Specific Details

### Service Management

Uses launchd. Service plist template: [config/ai.openclaw.gateway.plist](config/ai.openclaw.gateway.plist) (to be created).

Alternatively, OpenClaw's built-in `openclaw onboard --install-daemon` installs a launchd agent automatically.

### Firewall

macOS Application Firewall (`socketfilterfw`) or `pf` packet filter for advanced rules.

### User Isolation

Options:
- Run as a dedicated macOS user (created via `sysadminctl` or `dscl`).
- Run as the primary user's launchd agent with filesystem restrictions.

The dedicated user approach provides stronger isolation.

### Credential Storage

- `~/.openclaw/.env` or a secured system location.
- macOS Keychain is an option for credential storage (stronger than file-based).
- If using a dedicated user, credentials live in that user's home directory with `600` permissions.

### Advantages Over Pi

- **SSD storage**: no SD card failure risk, faster I/O.
- **More resources**: Apple Silicon Macs have significantly more RAM and CPU than a Pi.
- **FileVault**: built-in full-disk encryption (at rest).
- **Better thermal management**: fan-cooled (Mini) vs passive (Pi).
- **Native macOS node**: OpenClaw's macOS app can run as a node on the same machine — voice wake, canvas, camera, etc.

### Known Considerations

- **Power consumption**: Mac Mini uses more power than a Pi (not ideal for low-power scenarios).
- **Cost**: significantly more expensive than a Pi.
- **macOS updates**: major OS updates may require manual intervention. Auto-update for security patches only.
- **Headless operation**: Mac Mini can run headless but some macOS features assume a display.

## Common References

This platform extends the shared titanium documentation:

- [Identity Isolation](../../common/identity-isolation.md)
- [Security Model](../../common/security-model.md)
- [Credential Lifecycle](../../common/credential-lifecycle.md)
- [OpenClaw Install](../../common/openclaw-install.md)
- [Audit Framework](../../common/audit-framework.md)

## Status

Scaffolding only. Detailed playbooks and config templates will be developed after the Pi platform reaches M3 maturity.
