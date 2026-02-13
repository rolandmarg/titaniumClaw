# titanium Roadmap

## Cross-Platform Milestones

### Phase 1: Pi Platform (piClaw)

The Raspberry Pi is the first platform. All common docs are developed alongside it.

#### M0 - Preflight
- Flash Pi OS Lite 64-bit with SSH and public key pre-enabled (headless)
- Verify SSH connectivity
- Record device IP/hostname

#### M1 - Bootstrap
- Update OS packages
- Install prerequisites (git, curl, build-essential, ufw, openssl)
- Install Node.js 22
- Install OpenClaw
- Create openclaw system user and directories
- Generate gateway token, write env file with strict permissions

#### M2 - Deploy
- Install and enable systemd service for OpenClaw gateway
- Verify auto-restart behavior
- Smoke test health endpoint on loopback
- Confirm journald logging

#### M3 - Security Hardening
- Apply UFW firewall (default deny inbound, SSH only)
- Lock down SSH (key-only, no root, no password)
- Disable unused services
- Enable unattended security updates
- Verify loopback bind and token auth
- Enforce filesystem isolation for openclaw user
- Set up identity isolation (dedicated email, Discord bot, API accounts)
- Run full audit — all checks pass

#### M3b - YubiKey Hardening (optional)
- Configure SSH with FIDO2/U2F YubiKey authentication
- Require physical presence for SSH login
- Document enrollment, recovery, and fallback procedures

#### M4 - Reliability and Ops
- Health check systemd timer
- Automated daily backups with retention and encryption
- Restore procedure tested on clean image
- Upgrade strategy for Node.js and OpenClaw
- End-to-end: fresh SD card to fully operational in one session

### Phase 2: Mac Mini Platform (miniClaw)

Developed after Pi reaches M3+ maturity. Leverages common docs, adds macOS-specific:

- macOS bootstrap (Homebrew, Node.js, OpenClaw)
- launchd agent configuration
- macOS firewall / pf setup
- FileVault verification
- Remote Login (SSH) hardening
- macOS-specific audit checks
- OpenClaw macOS node integration (voice wake, canvas)

### Phase 3: Refinement

- Off-device backup strategy (all platforms)
- Tailscale integration playbook
- Incident response playbook
- Decommission/offboarding playbook
- Multi-device management (if needed)
