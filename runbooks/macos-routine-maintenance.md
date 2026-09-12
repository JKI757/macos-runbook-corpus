# macOS Routine Maintenance Runbook

## Use this when

- The Mac needs a routine health check.
- Storage is getting low.
- Updates or a restart have been deferred for a long time.
- You want to clean up safely without deleting system data blindly.

## 1. Capture a baseline

Record the macOS version, uptime, and available storage:

```sh
sw_vers
uptime
df -h /
```

Check for a severely full startup volume. Leave working room before attempting large updates or rebuilds.

## 2. Review obvious storage consumers

Use Finder or **System Settings → General → Storage** to review large categories. From the shell, inspect only locations you recognize:

```sh
du -sh "$HOME/Downloads" "$HOME/Desktop" 2>/dev/null
```

Delete or archive only files you have identified. Empty Trash after confirming the contents are no longer needed.

## 3. Apply updates deliberately

Open **System Settings → General → Software Update** and review the available update, required free space, restart requirement, and backup status before installing it.

Do not interrupt an update or force a shutdown while it is applying.

## 4. Restart when it is an appropriate repair

A normal restart can clear temporary process, memory, and service state. Save work first, then use the Apple menu or:

```sh
sudo shutdown -r now
```

Use the graphical restart when possible. Reserve the command for a deliberate, attended restart.

## 5. Verify after maintenance

After updates or a restart, confirm:

- The Mac reaches the normal login or desktop.
- Required applications and user services start.
- Available storage is still adequate.
- The original symptom is gone.

## Do not do this

- Do not delete `/System`, `/Library`, or unknown files under `~/Library` as a generic cleanup step.
- Do not install a cleanup utility solely because it reports caches or logs.
- Do not start an update without a current backup when the data is important.
