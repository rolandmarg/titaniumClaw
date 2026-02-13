# Credential Lifecycle

Every credential used by titanium follows a defined lifecycle: **create, store, rotate, revoke**. This document defines the process for each phase.

## Credential Inventory

| Credential | Purpose | Storage Location | Rotation Schedule |
|---|---|---|---|
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication | `openclaw.env` | Quarterly or on-demand |
| Service email password | OpenClaw's email identity | `identity.env` | Quarterly |
| Discord bot token | OpenClaw's Discord presence | `identity.env` | Quarterly or on compromise |
| Channel API keys | Telegram bot token, Slack tokens, etc. | `openclaw.env` or `identity.env` | Per provider recommendation |
| SSH host key | Pi/Mac SSH identity | `/etc/ssh/` or system keychain | Rarely (on re-provision) |
| SSH user key | Your access to the device | `~/.ssh/` on your Mac | On compromise or YubiKey change |

## Phase 1: Create

### Gateway Token

```bash
# Generate a strong random token
openssl rand -hex 32
```

### Service Email

- Create via provider API (Migadu, Fastmail, or self-hosted).
- Use a strong generated password (`openssl rand -base64 24`).
- Store credentials immediately (Phase 2).

### Discord Bot

- Register application at https://discord.com/developers/applications.
- Create bot user, generate token.
- Store token immediately (Phase 2).

### Channel Tokens

- Follow OpenClaw's channel setup: `openclaw channels login` for each channel.
- Store any generated tokens/keys in the appropriate env file.

## Phase 2: Store

### Rules

- All credentials stored in env files with **`600` permissions**, owned by the **service user**.
- Never committed to git. `.gitignore` must exclude `*.env` and any file containing secrets.
- Separate files for separate concerns:
  - `openclaw.env` — gateway operational config (token, bind, port).
  - `identity.env` — service identity credentials (email, Discord, API keys).

### Storage Path

| Platform | Env file location |
|---|---|
| Raspberry Pi (Linux) | `/etc/openclaw/openclaw.env`, `/etc/openclaw/identity.env` |
| Mac Mini (macOS) | `~/.openclaw/.env` or platform-appropriate secure location |

### Verification

```bash
# Check permissions and ownership (Linux)
stat -c '%a %U:%G' /etc/openclaw/openclaw.env
# Expected: 600 openclaw:openclaw

stat -c '%a %U:%G' /etc/openclaw/identity.env
# Expected: 600 openclaw:openclaw
```

## Phase 3: Rotate

### Pre-Rotation

1. **Back up** the current env file:
   ```bash
   sudo cp /etc/openclaw/openclaw.env /etc/openclaw/openclaw.env.bak.$(date +%Y%m%d)
   sudo chmod 600 /etc/openclaw/openclaw.env.bak.*
   ```

2. **Verify** the service is currently healthy before making changes.

### Rotation

3. **Generate** the new credential value.

4. **Update** the env file with the new value:
   ```bash
   # Example: rotate gateway token
   NEW_TOKEN=$(openssl rand -hex 32)
   sudo sed -i "s/OPENCLAW_GATEWAY_TOKEN=.*/OPENCLAW_GATEWAY_TOKEN=$NEW_TOKEN/" /etc/openclaw/openclaw.env
   ```

5. **Restart** the service:
   ```bash
   sudo systemctl restart openclaw-gateway  # Linux
   # or: launchctl kickstart -k ...         # macOS
   ```

### Post-Rotation

6. **Verify** the service is healthy after restart:
   ```bash
   # Check service status
   systemctl is-active openclaw-gateway

   # Check health endpoint
   curl -sf http://127.0.0.1:18789/health
   ```

7. **If health check fails**: roll back immediately:
   ```bash
   sudo cp /etc/openclaw/openclaw.env.bak.YYYYMMDD /etc/openclaw/openclaw.env
   sudo systemctl restart openclaw-gateway
   ```

8. **Clean up** old backup after confirming rotation success (keep for 7 days minimum).

## Phase 4: Revoke

Used when decommissioning the device or responding to a compromise.

### On Decommission

1. **Revoke Discord bot token**: via Discord Developer Portal or API — regenerate (invalidates old token).
2. **Delete or disable service email account**: via provider API or admin panel.
3. **Invalidate all channel tokens**: disconnect channels via `openclaw channels` or revoke at provider.
4. **Wipe the device**: re-flash SD card (Pi) or erase disk (Mac).
5. **Remove SSH authorized keys**: remove the device's host key from `~/.ssh/known_hosts` on your Mac.
6. **Update inventory**: remove the device from any documentation or asset tracking.

### On Compromise

1. **Immediately rotate all credentials** (gateway token, email password, Discord token, channel keys).
2. **Revoke the compromised credential** at the provider level (not just locally).
3. **Review logs** for unauthorized activity.
4. **Re-run the audit playbook** to verify system integrity.
5. **Document the incident** in the daily note or incident log.

## Audit

The audit framework checks:

- All env files exist with correct permissions (`600`, correct owner).
- No credentials appear in git history (`git log -p | grep -i token`).
- No credentials appear in system logs (`journalctl | grep -i token`).
- Rotation has occurred within the scheduled window.
- Backup env files are not older than the retention period.

See [audit-framework.md](audit-framework.md) for the full check structure.
