# Identity Isolation

OpenClaw runs as a **sandboxed service identity** — completely separate from your personal identity. This is a core security principle across all titanium platforms.

## Principle

OpenClaw sees only what it needs to operate. No personal data bleeds in.

## Boundaries

| Allowed | Not Allowed |
|---|---|
| Chat sessions and conversation history | Personal email credentials |
| Operational config (`openclaw.json`, `.env`) | Personal browser data or cookies |
| Service-owned API keys and tokens | Personal social media accounts |
| Service-owned email and Discord accounts | Personal contacts or address book |
| Workspace files (`~/.openclaw/workspace`) | Home directory files outside workspace |

## Service Accounts

OpenClaw gets its own accounts for any external integrations:

- **Email**: dedicated service email (not your personal address). Used for API signups, notifications, recovery.
- **Discord**: dedicated bot application with its own token. Registered via Discord Developer Portal.
- **Messaging channels**: each channel (WhatsApp, Telegram, Slack, etc.) should use a service-owned number/account where possible.
- **API keys**: any third-party API keys are service-specific, not tied to personal accounts.

## Credential Storage

All service credentials live in a secured env file on the host:

- **Path**: `/etc/openclaw/identity.env` (Linux) or platform-equivalent.
- **Permissions**: `600`, owned by the service user.
- **Contents**: email address, email password, Discord bot token, API keys.
- **Never** stored in git. `.env.example` shows the structure without values.

## Filesystem Isolation

The OpenClaw process should only access:

- Its data directory (e.g., `/var/lib/openclaw` on Linux, `~/Library/Application Support/OpenClaw` on macOS).
- Its config directory (e.g., `/etc/openclaw` on Linux).
- System libraries and the Node.js runtime.

Enforced via:

- **Linux**: dedicated system user (`openclaw`) with `nologin` shell. systemd sandboxing: `ProtectHome=true`, `ProtectSystem=full`, `NoNewPrivileges=true`, `PrivateTmp=true`.
- **macOS**: dedicated user or launchd sandboxing with restricted filesystem access.

## Account Provisioning

Where possible, automate account creation via provider APIs:

1. **Email**: pick a provider with API support (Migadu, Fastmail, or self-hosted). Script account creation and credential storage.
2. **Discord**: register a bot application via Discord API. Store bot token in `identity.env`.
3. **Other channels**: follow OpenClaw's channel-specific setup (`openclaw channels login`) using the service identity.

## Credential Rotation

Credentials should be rotatable:

1. Back up current credentials before rotation.
2. Generate new credential via provider API or CLI.
3. Update the env file on the host.
4. Restart the OpenClaw service.
5. Verify health after restart.
6. If health check fails, roll back from backup.

See [credential-lifecycle.md](credential-lifecycle.md) for the full lifecycle.

## Platform-Specific Notes

- **Raspberry Pi**: see [platforms/pi/](../platforms/pi/) for Linux user isolation and systemd hardening.
- **Mac Mini**: see [platforms/macmini/](../platforms/macmini/) for macOS user isolation and launchd sandboxing.
