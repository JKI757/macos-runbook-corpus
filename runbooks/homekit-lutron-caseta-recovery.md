# HomeKit / Lutron Caséta Recovery

## Use this when

- Lutron lights work in the Lutron app but fail in Apple Home or Home scenes.
- A HomePod or Apple TV reports online, yet one manufacturer's accessories fail.
- You need to separate network discovery, local connectivity, and authenticated HomeKit control.

## Safety and scope

Start read-only. Do not remove the Lutron bridge, reset HomeKit, hold its rear button, or recreate the Home as an initial repair. Removing/resetting the integration can remove Lutron accessories and their automation associations. Do not restart the whole network when a narrower test is available. Do not sign into or change an Apple Account without authorization.

## 1. Establish the failure boundary

1. Test one affected light in the Lutron app, then in Apple Home. Record the exact Home error and whether scenes also fail.
2. Check whether accessories from other manufacturers still work in Home.
3. Test Apple Home with the phone on the bridge's actual LAN; distinguish local failures from remote-only failures.
4. Record the active hub in **Home → … → Home Settings → Home Hubs & Bridges**. Merely seeing all hubs online is not enough: identify the active hub at the top.
5. Record pending updates for each hub without assuming an update caused a failover.

If the Lutron app also fails, investigate the bridge/network/Lutron side first. If only Apple Home fails, prioritize its integration and active hub.

## 2. Separate Bonjour discovery from connectivity

On the diagnostic Mac, identify the Wi-Fi interface and current addressing:

```sh
networksetup -listallhardwareports
route -n get default
ifconfig
```

Browse for accessories, then resolve the exact advertised bridge instance (stop each long-running command with Control-C):

```sh
dns-sd -B _hap._tcp local.
dns-sd -L '<exact bridge instance>' _hap._tcp local.
dns-sd -B _airplay._tcp local.
dns-sd -L '<exact hub instance>' _airplay._tcp local.
```

Use the resolved address and advertised port, not hardcoded examples:

```sh
route -n get <bridge-ip>
nc -vz -G 3 <bridge-ip> <advertised-homekit-port>
```

Interpretation:

- Bonjour/mDNS advertisements may be reflected between VLANs even when TCP control traffic is blocked.
- A timeout from another VLAN does not prove the bridge or HomePod is offline. Repeat on the accessory's LAN. A multihomed Mac's default route may not be its Wi-Fi route; check the route to the exact target.
- An open HomeKit TCP port proves transport reachability, not pairing health or successful commands.
- A paired bridge may advertise `sf=0`. An unauthenticated diagnostic GET to `/accessories` may return HTTP 470/authentication required. Neither result alone diagnoses corrupt pairing.
- AirPlay advertisements and advertised software versions do not establish which device is the active Home hub or whether its HomeKit sessions are healthy.
- macOS can redact SSIDs or misreport association without appropriate permissions; verify addressing and the Wi-Fi UI rather than trusting a single CLI message.

## 3. Switch the active Apple Home hub before resetting pairings

When the Lutron app works and local HomeKit transport is reachable:

1. In **Home Hubs & Bridges**, record the current active hub and Automatic Selection setting.
2. Turn off **Automatic Selection**, if available, and select a different available HomePod or Apple TV as preferred hub.
3. Wait until Home confirms that the new hub is active.
4. Test the same Lutron light and affected scene.
5. If control returns, leave the working hub selected during observation. Record the old/new hub roles and actual test result privately; do not publish household names or addresses.

Apple documents preferred-hub selection in [its home-hub guide](https://support.apple.com/en-us/102557).

If the selector is unavailable, record the UI/software versions and use Apple's documented hub restart procedure rather than inventing controls.

**Interpretation:** recovery after switching hubs implicates the previous hub's HomeKit path/session/state, but does not prove a hardware failure, a specific software defect, or the reason the previous hub became active. Online and up-to-date status do not substitute for a successful accessory command.

## 4. Recover and maintain the previously active hub

After restoring service, restart the previously failing hub using its supported restart control, or a power-only cycle. Apply pending hub updates during a suitable maintenance window, one device at a time, keeping another working hub available. Updates/restarts can temporarily interrupt service; verify afterward.

A pending update on a formerly preferred Apple TV is a clue to investigate, not proof that the update caused automatic hub selection to move elsewhere. Establishing the reason requires contemporaneous hub/update history that may not be available.

Do not immediately switch back just to erase the symptom. If a controlled retest is wanted, record the known-good preference, select the recovered hub, test the same light/scene, and return to the known-good hub if failure recurs.

## 5. If switching hubs does not help

- Try a power-only restart of the Lutron bridge: unplug power for 30 seconds, reconnect, wait for startup, and retest. Leave Ethernet connected; do not hold the rear button.
- Follow Apple's accessory troubleshooting guidance for restarting bridges/home hubs and checking updates.
- Review network changes, bridge/hub LAN membership, isolation, and firewall policy. A working cloud-connected accessory does not prove local HomeKit connectivity is allowed.
- Consult Lutron support before a destructive HomeKit re-pair. Document rooms, scenes, and automations and obtain explicit approval before removing/resetting the integration.

## Observed recovery pattern (sanitized)

- Lutron app controlled lights successfully; Apple Home/scenes did not.
- Bridge and Apple hubs were discoverable across networks, but initial cross-network TCP tests timed out.
- Joining the bridge's LAN made the bridge HomeKit port and HomePod AirPlay ports reachable.
- Power-cycling the Lutron bridge did not restore Apple Home control.
- The active HomePod appeared online and current. Selecting another HomePod restored scenes, as confirmed by the operator.
- An Apple TV thought to have previously been active showed a pending update. The timing/reason for the earlier hub change was not established.

This demonstrates a useful troubleshooting branch, not a universal diagnosis. Recovery was operator-confirmed; no authenticated Home control was available on the diagnostic Mac because its Home app required an iCloud sign-in.

## Verification checklist

- [ ] One affected light responds in Lutron and Apple Home.
- [ ] Previously failing scenes run successfully.
- [ ] Active/preferred hub and Automatic Selection setting are recorded.
- [ ] If remote control matters, test off Wi-Fi separately.
- [ ] If automations matter, verify an actual scheduled/triggered run separately; manual scene success does not prove automation execution.
- [ ] Pending updates/restarts and any untested rollback are explicitly noted.
- [ ] No pairing reset or automation loss occurred during the narrow recovery.

## Vendor references

- [Apple: Set up and select a home hub](https://support.apple.com/en-us/102557)
- [Apple: HomeKit or Matter accessory not responding](https://support.apple.com/en-us/102056)
- [Lutron: Multiple Caséta devices not responding in Apple Home](https://support.lutron.com/us/en/product/casetawireless/article/troubleshooting/Why-are-all-my-Lutron-devices-showing-no-response-in-the-Apple-Home-app)
