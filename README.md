# Local Runbook Corpus

A small, local-first corpus of operational runbooks for macOS troubleshooting and routine maintenance.

## Layout

- `runbooks/` — task-oriented Markdown instructions.
- `manifest.json` — machine-readable document inventory and tags.

## Operating rules

1. Start with read-only checks.
2. Record the exact error and the command output before changing state.
3. Prefer the narrowest repair that addresses the observed failure.
4. Verify the result after every repair.
5. Stop before destructive or irreversible actions unless the scope is explicit and a backup exists.
6. Treat external-volume and network-service errors separately from the startup-volume or local-service result being tested.

## Included runbooks

- [Spotlight Recovery](runbooks/spotlight-recovery.md) — restore application indexing and repair the Spotlight UI.
- [macOS Routine Maintenance](runbooks/macos-routine-maintenance.md) — safe health checks, updates, storage review, and restart guidance.
- [Network Connectivity Basics](runbooks/network-connectivity-basics.md) — identify whether a failure is local, gateway, DNS, or service-specific.
- [launchd User Service Recovery](runbooks/launchd-user-service-recovery.md) — inspect and safely restart a known user-level launchd service.
- [Time Machine Backup Check](runbooks/time-machine-backup-check.md) — verify backup status and investigate common backup failures.

These runbooks are operational guidance, not a substitute for a current backup or vendor documentation.
