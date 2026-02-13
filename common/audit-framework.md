# Audit Framework

The audit framework defines a standard set of pass/fail checks that verify the security and operational state of any titaniumClaw deployment. Each platform extends the common checks with platform-specific ones.

## How Audits Work

1. An agent (or human) runs the audit checks via SSH or locally.
2. Each check outputs `PASS`, `FAIL`, `WARN`, or `SKIP`.
3. Results are collected and reported.
4. Any `FAIL` result is logged as a blocker.
5. `SKIP` results are acceptable for uncompleted milestones.
6. `WARN` results should be tracked for follow-up.

## Check Format

Every check follows this pattern:

```bash
# Description of what we're checking
ACTUAL=$(command to get actual state)
EXPECTED="expected value"
if [ "$ACTUAL" = "$EXPECTED" ]; then
  echo "PASS: description ($ACTUAL)"
else
  echo "FAIL: description (expected $EXPECTED, got $ACTUAL)"
fi
```

## Common Checks (all platforms)

### OpenClaw Runtime

| Check | Command | Expected |
|---|---|---|
| Node.js version | `node --version` | `v22.x.x` |
| OpenClaw installed | `openclaw --version` | Returns version string |
| OpenClaw config exists | `test -f ~/.openclaw/openclaw.json` | File exists |
| Workspace exists | `test -d ~/.openclaw/workspace` | Directory exists |

### Gateway

| Check | Command | Expected |
|---|---|---|
| Gateway service running | Platform-specific service check | Active/running |
| Gateway on loopback only | Check listening address for port 18789 | `127.0.0.1` only, NOT `0.0.0.0` |
| Gateway token configured | Check env file contains `OPENCLAW_GATEWAY_TOKEN` | Non-empty value |
| Health endpoint responds | `curl -sf http://127.0.0.1:18789/health` | HTTP 200 or valid response |

### Credentials

| Check | Command | Expected |
|---|---|---|
| Env file permissions | Check file mode | `600` |
| Env file ownership | Check file owner | Service user |
| No secrets in git | `git log -p \| grep -ci "token\|password\|secret"` | 0 matches (or only in .example files) |
| No secrets in logs | Check recent logs for credential patterns | 0 matches |
| Identity env exists (if M3 done) | Check identity.env file | File exists with `600` perms |

### Network

| Check | Command | Expected |
|---|---|---|
| Firewall active | Platform-specific firewall check | Active with default deny inbound |
| SSH listening | Check SSH port | Listening |
| No unexpected listeners | List all listening ports | Only expected services |

### SSH

| Check | Command | Expected |
|---|---|---|
| Root login disabled | Check sshd config | `PermitRootLogin no` |
| Password auth disabled | Check sshd config | `PasswordAuthentication no` |
| Pubkey auth enabled | Check sshd config | `PubkeyAuthentication yes` |

### Identity Isolation

| Check | Command | Expected |
|---|---|---|
| Service user exists | Check user database | User exists with nologin shell |
| Service user cannot read personal dirs | Test access to home dirs | Permission denied |
| Identity env secured | Check identity.env permissions | `600`, service user owned |

## Platform-Specific Checks

### Raspberry Pi (Linux)

| Check | What |
|---|---|
| systemd service enabled and active | `systemctl is-enabled && is-active` |
| systemd sandboxing | `ProtectHome`, `ProtectSystem`, `NoNewPrivileges`, `PrivateTmp` |
| UFW active with default deny | `ufw status verbose` |
| Unattended upgrades enabled | Check apt config |
| OS is 64-bit ARM | `uname -m` = `aarch64` |
| NTP synchronized | `timedatectl` |

### Mac Mini (macOS)

| Check | What |
|---|---|
| launchd agent loaded and running | `launchctl list \| grep openclaw` |
| macOS firewall enabled | `socketfilterfw --getglobalstate` |
| Remote Login restricted | Check SSH access controls |
| FileVault enabled | `fdesetup status` |
| Automatic updates enabled | `softwareupdate --schedule` |

## Audit Levels

| Level | When to Run | What's Checked |
|---|---|---|
| Quick | After any single change | Just the changed component |
| Milestone | After completing M1/M2/M3/M4 | All checks relevant to that milestone |
| Full | Periodically or on-demand | Every check in the framework |

## Reporting

After an audit, the agent (or human) should:

1. **Report** the summary: total PASS/FAIL/WARN/SKIP counts.
2. **Log** any FAIL results as blockers (in open-loops or issue tracker).
3. **Record** the audit completion date in the backlog or daily note.
4. **Follow up** on WARN results within 7 days.
