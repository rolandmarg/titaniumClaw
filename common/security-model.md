# Security Model

## Warning

OpenClaw and agentic AI systems are powerful but immature. Security guardrails, sandboxing, and safety mechanisms are not yet robust across the ecosystem. titaniumClaw attempts to harden the deployment surface, but it cannot guarantee complete protection against all threat vectors — including ones introduced by the AI agent itself.

**Use common sense. Review what the agent does. Limit network exposure. Do not trust any AI system with unsupervised access to sensitive data or critical infrastructure without understanding the risks.**

## Threat Model

### Threat Actors

| Actor | Description | Likelihood |
|---|---|---|
| Network-adjacent attacker | Someone on the same LAN scanning for open ports | Medium |
| Remote attacker | Internet-based scanning/exploitation | Low (gateway is loopback-only) |
| Compromised AI agent | The OpenClaw agent itself acting outside expected boundaries | Medium |
| Compromised dependency | Supply chain attack via Node.js packages or OpenClaw updates | Low-Medium |
| Physical access attacker | Someone with physical access to the device | Low |
| Credential leakage | Secrets exposed via logs, backups, or process listings | Medium |

### Attack Vectors and Mitigations

| Vector | Mitigation |
|---|---|
| Gateway exposed to network | Bind to loopback only. Access via SSH tunnel or Tailscale. |
| Unauthenticated gateway access | Strong random gateway token. OpenClaw DM pairing for messaging. |
| SSH brute force | Key-only auth, no password, no root login. Optional YubiKey FIDO2. Rate limiting / fail2ban. |
| Unnecessary services running | Disable unused services. Minimal OS install (no desktop). |
| Unpatched vulnerabilities | Unattended security updates enabled. |
| Agent reads personal data | Filesystem isolation via OS user permissions + service manager sandboxing. |
| Agent uses personal accounts | Identity isolation — OpenClaw has its own email, Discord, API accounts. |
| Secrets in git | `.gitignore` excludes `.env`. Secrets stored outside repo with `600` perms. |
| Secrets in logs | Configure log level to avoid token/credential output. Review journald/syslog. |
| Secrets in backups | Encrypt backups. Restrict backup file permissions. |
| SD card / disk theft | Full-disk encryption where supported. Off-device backup for recovery. |
| curl-pipe-bash installs | Download first, inspect, then execute. Pin versions. Verify checksums where available. |
| Unattended updates break OpenClaw | Restrict auto-updates to security patches only. Pin Node.js version. Post-update health check. |

### What We Explicitly Do NOT Protect Against

- A fully compromised OpenClaw process with root access (defense in depth reduces but doesn't eliminate this).
- Physical attack with unlimited time and resources.
- Zero-day vulnerabilities in the Linux kernel, Node.js, or OpenClaw itself.

The goal is defense in depth with a small, auditable surface — not perfect security.

## Security Principles

1. **Loopback by default**: the gateway never binds to a public interface. Expose only via SSH tunnel or Tailscale Serve/Funnel.
2. **Least privilege**: the OpenClaw process runs as a dedicated user with no login shell and minimal filesystem access.
3. **Token auth**: every gateway connection requires a strong random token.
4. **DM pairing**: OpenClaw's built-in sender verification requires unknown senders to be explicitly approved.
5. **Secrets outside git**: all credentials stored in env files with `600` permissions, never committed.
6. **Identity isolation**: OpenClaw uses its own service accounts, never personal credentials.
7. **Minimal surface**: lean OS, no desktop environment, unused services disabled, strict firewall.
8. **Auditability**: all security controls are verifiable via the audit framework.
9. **Defense in depth**: multiple overlapping controls (firewall + bind policy + token auth + user isolation + systemd sandboxing).
10. **Fail closed**: when in doubt, deny access. Default deny inbound. Default reject unknown senders.

## Network Security

### Default (loopback-only)

```
[Internet] --X-- [Pi/Mac firewall] --X-- [Gateway on 127.0.0.1:18789]
                                              |
[Your Mac] ---SSH tunnel---> [127.0.0.1:18789]
```

No inbound ports open except SSH (key-only).

### With Tailscale (future, optional)

```
[Internet] --X-- [Firewall]
                     |
[Tailnet peers] --- [Tailscale Serve] --- [Gateway on 127.0.0.1:18789]
```

Gateway stays loopback. Tailscale handles encrypted mesh access.

## Platform-Specific Security

Each platform adds its own controls on top of this common model:

- **Raspberry Pi**: UFW firewall, `sshd_config` hardening, systemd sandboxing, unattended-upgrades. See [platforms/pi/](../platforms/pi/).
- **Mac Mini**: macOS firewall / pf, Remote Login restrictions, launchd sandboxing. See [platforms/macmini/](../platforms/macmini/).

## Optional: YubiKey Hardening

For SSH access, a YubiKey adds hardware-bound authentication:

- `ssh-keygen -t ed25519-sk -O resident -O verify-required` creates a key that requires physical touch.
- SSH login fails without the YubiKey present.
- Recommended for high-security deployments or devices on less-trusted networks.
- Requires a documented fallback path (encrypted recovery key stored offline).
