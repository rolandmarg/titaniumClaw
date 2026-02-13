# OpenClaw Install (Platform-Agnostic)

This document covers the parts of OpenClaw installation that are the same across all platforms. Platform-specific prerequisites (OS packages, service managers) are in the platform directories.

## Upstream Reference

| Field | Value |
|---|---|
| Name | OpenClaw |
| GitHub | https://github.com/openclaw/openclaw |
| Website | https://openclaw.ai |
| Docs | https://docs.openclaw.ai |
| License | MIT |
| Runtime | Node.js >= 22 |
| Default gateway port | 18789 |
| Default gateway bind | loopback (`ws://127.0.0.1:18789`) |

## Prerequisites

- **Node.js >= 22** (install method varies by platform — see platform docs).
- **npm** or **pnpm** (pnpm preferred for source builds).
- **git** (for source installs or updates).

## Install Methods

### Method 1: npm global install (recommended)

```bash
npm install -g openclaw@latest
# or:
pnpm add -g openclaw@latest
```

**Verify**: `openclaw --version` returns the installed version.

### Method 2: From source (development / hackable)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
# Run via: pnpm openclaw ...
```

## Onboarding

After install, run the onboarding wizard:

```bash
openclaw onboard --install-daemon
```

The wizard guides through:
1. Gateway configuration (port, bind, auth).
2. Workspace setup (`~/.openclaw/workspace`).
3. Channel connections (WhatsApp, Telegram, Slack, Discord, etc.).
4. Skills configuration.

The `--install-daemon` flag installs:
- **Linux**: a systemd user service.
- **macOS**: a launchd agent.

This keeps the gateway running across reboots.

## Configuration

Config file: `~/.openclaw/openclaw.json`

Minimal config:

```json
{
  "agent": {
    "model": "anthropic/claude-opus-4-6"
  }
}
```

Full config reference: https://docs.openclaw.ai/gateway/configuration

## Key CLI Commands

| Command | Purpose |
|---|---|
| `openclaw onboard` | Guided setup wizard |
| `openclaw gateway --port 18789 --verbose` | Start gateway manually |
| `openclaw doctor` | Health check and migration diagnostics |
| `openclaw update --channel stable` | Update OpenClaw (`stable`, `beta`, or `dev`) |
| `openclaw channels login` | Link messaging channels |
| `openclaw agent --message "..."` | Send a message to the agent |
| `openclaw pairing approve <channel> <code>` | Approve a DM sender |
| `openclaw nodes` | Manage device nodes (macOS/iOS/Android) |

## Security Defaults

- **DM pairing** enabled by default — unknown senders must be approved before the bot processes their messages.
- **Loopback bind** by default — gateway only listens on `127.0.0.1`.
- Run `openclaw doctor` to surface risky or misconfigured DM policies.

See [security-model.md](security-model.md) for the full titanium security model.

## Workspace Structure

```
~/.openclaw/
  openclaw.json              # Main config
  credentials/               # Channel credentials (auto-managed)
  workspace/
    AGENTS.md                # Injected agent prompt
    SOUL.md                  # Agent personality
    TOOLS.md                 # Tool configuration
    skills/                  # Installed skills
      <skill>/SKILL.md
```

## Updating

```bash
openclaw update --channel stable
# or manually:
npm install -g openclaw@latest
```

After updating, run `openclaw doctor` to check for migrations.

## Platform-Specific Notes

- **Raspberry Pi**: Node.js installed via nodesource. Service runs as system-level systemd unit under dedicated `openclaw` user. See [platforms/pi/](../platforms/pi/).
- **Mac Mini**: Node.js installed via Homebrew or nvm. Service runs as launchd user agent. See [platforms/macmini/](../platforms/macmini/).
