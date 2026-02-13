# Hardening Checklist

## Host Baseline

- [ ] Update OS packages: `sudo apt update && sudo apt upgrade -y`
- [ ] Enable unattended security updates
- [ ] Set correct timezone and NTP sync
- [ ] Remove/disable unused packages and services

## SSH and Access

- [ ] Create non-root admin user
- [ ] Disable SSH password auth
- [ ] Disable root SSH login
- [ ] Enforce SSH key auth only

## Network Controls

- [ ] Default deny inbound policy
- [ ] Allow only required ports (usually SSH)
- [ ] Keep OpenClaw gateway loopback-bound unless explicitly needed
- [ ] If exposed, protect with reverse proxy + TLS + strict firewall

## OpenClaw Service

- [ ] Run as dedicated low-privilege service user
- [ ] Set restart policy (`Restart=always`)
- [ ] Configure token auth with high-entropy secret
- [ ] Restrict config and env files to `600`

## Monitoring and Recovery

- [ ] Verify service health after reboot
- [ ] Capture logs via journald and retention policy
- [ ] Define backup for OpenClaw config and workspace
- [ ] Test restore path on a clean image
