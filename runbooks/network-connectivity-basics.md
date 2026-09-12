# Network Connectivity Basics Runbook

## Use this when

- The Mac cannot reach a local service or the Internet.
- Wi-Fi appears connected but applications report no network.
- You need to separate a device, gateway, DNS, or remote-service failure.

## 1. Identify the active interface and address

```sh
route -n get default
ifconfig
```

Record the default interface, local IP address, and default gateway. Do not change network settings until these values are known.

## 2. Test the local network path

Ping the default gateway shown by `route -n get default`:

```sh
ping -c 4 <default-gateway>
```

If the gateway is unreachable, inspect Wi-Fi association, Ethernet cabling, VLAN assignment, and the local router or access point before investigating DNS.

## 3. Test name resolution separately

```sh
scutil --dns
nslookup example.com
```

If the gateway responds but name resolution fails, inspect the active DNS configuration or test the configured resolver. Avoid changing DNS globally as a first step.

## 4. Renew the local connection

For a Wi-Fi connection, toggle Wi-Fi off and on from Control Center, then retest. For Ethernet, reseat the cable or reconnect the interface. Reboot the router only when you own it or have permission and have checked for other users.

## 5. Verify the actual service

A successful ping does not prove that an application service is healthy. Test the specific service using its documented client or a read-only health check, and record the exact error and timestamp.

## Do not do this

- Do not repeatedly delete network preference files without recording the current configuration.
- Do not assume an Internet outage when only DNS or one remote service is failing.
- Do not expose credentials, tokens, or private hostnames in a support ticket or shared log.
