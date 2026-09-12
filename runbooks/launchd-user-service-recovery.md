# launchd User Service Recovery Runbook

## Use this when

- A known user-level background service is not running.
- A local application depends on a service that stopped responding.
- You need to inspect launchd state before restarting anything.

## 1. Confirm the exact service label

Use the documented label for the service. List matching user agents:

```sh
launchctl list | grep -i '<service-name>'
```

If there is no match, inspect the service's installation documentation and `~/Library/LaunchAgents` before attempting a repair.

## 2. Inspect state and recent logs

```sh
launchctl print "gui/$(id -u)/<label>"
log show --last 15m --style compact --predicate 'process == "<process-name>"'
```

Check the last exit status, program path, environment assumptions, and repeated crash or permission errors.

## 3. Restart only the known user service

For a service already registered with launchd:

```sh
launchctl kickstart -k "gui/$(id -u)/<label>"
```

This stops and starts that specific service. Use `launchctl bootout` or remove a plist only when the service's documentation explicitly calls for it.

## 4. Verify recovery

```sh
launchctl print "gui/$(id -u)/<label>"
```

Then exercise the dependent application or local health endpoint and confirm that the service remains running for several minutes.

## Do not do this

- Do not unload every LaunchAgent as a generic fix.
- Do not use `kill -9` before inspecting why the service is failing.
- Do not edit a vendor plist in place without preserving a copy and knowing how launchd will reload it.
