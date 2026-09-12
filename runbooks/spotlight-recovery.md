# Spotlight Recovery Runbook

## Use this when

- Spotlight does not show applications.
- Application search returns no results.
- `mdutil` reports that the Spotlight server is disabled.

## 1. Check Spotlight state

```sh
mdutil -s /
```

If the result is `Indexing enabled`, go to step 3.

If the result is `Spotlight server is disabled`, go to step 2.

## 2. Enable Spotlight

Run:

```sh
sudo mdutil -a -i on
mdutil -s /
```

Enter the administrator password when asked.

Continue only when `/` reports:

```text
Indexing enabled
```

Errors for `/System/Volumes/Data` can occur. The important result is the status for `/`.

If Spotlight disables itself again, disconnect the Time Machine destination, restart the Mac, and repeat this step. A failed network Time Machine destination can trigger this problem.

## 3. Rebuild the index

After Spotlight is enabled, run:

```sh
sudo mdutil -E /
```

Keep the Mac awake and connected to power. Do not repeat this command during the rebuild.

## 4. Verify application indexing

Run:

```sh
mdfind 'kMDItemContentType == "com.apple.application-bundle"' | head
```

Successful output contains paths such as:

```text
/Applications/Safari.app
```

## 5. Repair the Spotlight user interface

If the command finds applications but Spotlight does not show them:

1. Open **System Settings → Spotlight**.
2. Turn **Applications** off and then on under Search Results.
3. Run:

```sh
killall Spotlight
```

Open Spotlight again with `Command-Space` and search for an application.

## Do not do this

- Do not delete `.Spotlight-V100` manually.
- Do not repeatedly erase the index.
- Do not treat external or network-volume indexing errors as a failure of the startup-volume index.
