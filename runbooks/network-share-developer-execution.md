# Running Development Builds from an SMB Share on macOS

## Use this when

- A Mach-O executable is owner-executable but returns `permission denied` or exit status `126` from an SMB-mounted share.
- The same binary runs after copying to a local APFS path such as `/tmp`.
- You need to run trusted development builds from a network share without disabling Gatekeeper globally.

## 1. Establish whether the failure is path-specific

Run the exact shared binary, then copy that same file to a local temporary path and run it again:

```sh
bin=/Volumes/<share>/<path-to-binary>
"$bin"; printf 'shared exit=%s\n' "$?"

cp "$bin" /tmp/macos-smb-exec-probe
chmod 700 /tmp/macos-smb-exec-probe
/tmp/macos-smb-exec-probe; printf 'local exit=%s\n' "$?"
```

If the local copy works and the SMB copy fails, do **not** conclude that the file mode is wrong merely because the share does not display `noexec`.

## 2. Capture policy evidence before changing settings

```sh
log show --last 1m --style compact \
  --predicate '(process == "syspolicyd") OR (process == "amfid") OR (process == "taskgated") OR (process == "kernel")'
```

Gatekeeper/XProtect network-location enforcement is indicated by entries such as:

- `com.apple.security.syspolicy.exec`
- `GK evaluateScanResult`
- XProtect evaluation
- notarization evaluation

This is distinct from an SMB POSIX-mode or mount-flag failure.

## 3. Enable the narrow developer-terminal exception

For trusted local development, register Terminal as a Developer Tool:

```sh
sudo spctl developer-mode enable-terminal
```

Then open **System Settings → Privacy & Security → Developer Tools** and enable the terminal application that launches the build. If you use a third-party terminal, add or enable that app instead of assuming Apple Terminal is the parent process.

Restart that terminal, rerun the exact shared binary, and verify that it starts successfully.

This permits development execution from that terminal; it does **not** disable Gatekeeper globally for Finder, downloads, or unrelated applications.

## 4. Use the appropriate distribution path

Developer Tools is for trusted local development. For software that must run from a network share for other users or outside a developer terminal, ship a Developer ID-signed and notarized app bundle, including every bundled dynamic library.

## Do not do this

- Do not disable Gatekeeper globally as the first response.
- Do not repeatedly use `chmod` or remove extended attributes without first comparing the SMB and local execution results.
- Do not treat `spctl --assess` rejection by itself as proof that a local development build cannot run; test the actual executable path.
- Do not claim an SMB share is `noexec` without evidence from the mount and the local-copy comparison.
