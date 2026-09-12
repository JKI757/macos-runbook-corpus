# Time Machine Backup Check Runbook

## Use this when

- You need to confirm that backups are running.
- Time Machine reports a failure or remains stuck.
- A backup destination may be unavailable or full.

## 1. Check current backup activity

```sh
tmutil status
```

Record whether a backup is running, the destination, and any reported error.

## 2. Confirm the destination is available

Open **System Settings → General → Time Machine** and confirm that the expected destination is listed and connected. For a network destination, verify the Mac can reach the storage host before changing Time Machine settings.

## 3. Check the latest completed backup

```sh
tmutil latestbackup
```

If no completed backup is reported, do not assume that a mounted destination contains a valid backup.

## 4. Start an attended backup

After confirming the destination and free space:

```sh
tmutil startbackup --auto
```

Keep the Mac connected to power and do not disconnect the destination while the backup is running. Recheck status:

```sh
tmutil status
```

## 5. Verify completion

Run `tmutil latestbackup` again after the backup finishes. Confirm that the returned path or timestamp is newer than the previous completed backup.

## Do not do this

- Do not erase or reformat a backup destination as a first troubleshooting step.
- Do not disconnect a destination during an active backup.
- Do not treat a failed network destination as proof that local startup-volume services are broken.
