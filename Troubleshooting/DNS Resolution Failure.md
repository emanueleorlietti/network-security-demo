---
Case: DNS Resolution Failure
Description: "A simulated lab case where an incorrect client DNS setting prevents access to an internal HTTPS service."
Status: Resolved
Date: 2026-10-12
tags:
  - troubleshooting
  - networking
  - dns
  - linux
  - lab
---

# 🚨 Problem

A Linux client cannot open `https://portal.demo.test/`.

The browser reports a name-resolution error. The objective is to determine whether the failure concerns DNS, network connectivity or the application itself.

> [!note] Simulated case
> This is an educational lab scenario, not a production incident. Configuration, command excerpts and results are illustrative. Reproducing the case requires the lab services described below.

# 🩺 Symptoms

- Access through the application hostname fails.
- The client has the expected IPv4 address and subnet prefix.
- ICMP replies arrive from the application server.
- DNS queries to the configured resolver time out.
- Queries to the intended resolver return the expected address.
- HTTPS succeeds when DNS is bypassed while preserving the hostname.

# 🖥️ Environment

| Component | Configuration |
|---|---|
| Operating system | Linux with NetworkManager |
| Active connection profile | `Demo LAN` |
| Network interface | `eth0` |
| Client address | `192.168.10.34/27`, statically configured |
| Default gateway | `192.168.10.33` |
| Intended DNS resolver | `192.168.10.53` |
| Incorrect configured resolver | `192.168.10.54` |
| Application hostname | `portal.demo.test` |
| Application server | `192.168.10.50` |
| Application service | HTTPS over TCP port `443` |

Lab assumptions:

- NetworkManager manages the interface and DNS configuration.
- No local caching resolver or VPN overrides the profile's DNS settings.
- `/etc/resolv.conf` reflects the active profile's resolver.
- `192.168.10.53` serves the lab DNS zone.
- No device responds at `192.168.10.54`.
- The HTTPS server presents a certificate trusted by the client and valid for `portal.demo.test`.
- The application returns HTTP `200` at `/`.
- All relevant devices are on the same local subnet.

The `.test` name is reserved for testing and is not a public endpoint.

# 🔍 Investigation

## 1. Confirm the Reported Failure

```bash
curl --verbose --connect-timeout 5 --max-time 15 --output /dev/null https://portal.demo.test/
```

Simulated error excerpt:

```text
curl: (6) Could not resolve host: portal.demo.test
```

The failure occurs before a connection to the application server is established.

This identifies name resolution as the immediate failure, but does not yet establish its underlying cause.

## 2. Inspect Local Addressing and Routing

```bash
ip -brief address show dev eth0
ip -4 route show
ip route get 192.168.10.50
```

Simulated excerpts:

```text
eth0             UP             192.168.10.34/27
```

```text
default via 192.168.10.33 dev eth0
192.168.10.32/27 dev eth0 proto kernel scope link src 192.168.10.34
```

```text
192.168.10.50 dev eth0 src 192.168.10.34 uid 1000
    cache
```

The address and prefix match the intended configuration. The server is directly connected, so traffic to it does not require the default gateway.

Route selection is correct for the supplied destination. This alone does not establish reachability.

## 3. Test IP Reachability

```bash
ping -4 -n -c 4 -W 2 192.168.10.50
```

Simulated summary:

```text
4 packets transmitted, 4 received, 0% packet loss
```

ICMP communication with the server succeeds during the test.

This supports local IP reachability, but does not prove that HTTPS works.

## 4. Inspect the Configured Resolver

```bash
nmcli -g ipv4.dns connection show "Demo LAN"
cat /etc/resolv.conf
```

Simulated results:

```text
192.168.10.54
```

```text
nameserver 192.168.10.54
```

The active configuration differs from the intended resolver address, `192.168.10.53`.

## 5. Compare Resolver Behavior

Query the configured resolver:

```bash
dig @192.168.10.54 portal.demo.test. A +time=2 +tries=1
```

Simulated outcome: the query times out and reports that no server could be reached.

Query the intended resolver:

```bash
dig @192.168.10.53 portal.demo.test. A +time=2 +tries=1
```

Simulated response excerpts:

```text
status: NOERROR
```

```text
;; ANSWER SECTION:
portal.demo.test.  300  IN  A  192.168.10.50
```

```text
;; SERVER: 192.168.10.53#53(192.168.10.53) (UDP)
```

Confirm TCP behavior:

```bash
dig @192.168.10.53 portal.demo.test. A +tcp +time=2 +tries=1
```

Simulated outcome: the same expected `A` record is returned over TCP.

The intended resolver answers the query over both transports. The configured resolver does not respond.

A timeout alone would not prove a configuration typo. The mismatch with the documented lab configuration supplies the additional evidence.

## 6. Test HTTPS Without Depending on DNS

```bash
curl --verbose --connect-timeout 5 --max-time 15 --noproxy portal.demo.test --resolve portal.demo.test:443:192.168.10.50 --output /dev/null https://portal.demo.test/
```

Simulated outcomes:

- The TCP connection to `192.168.10.50:443` succeeds.
- Certificate validation succeeds for `portal.demo.test`.
- The application returns HTTP `200`.

`--resolve` supplies the destination address while preserving the hostname for TLS and HTTP.

This confirms that the application's direct path works independently of the failing DNS lookup.

# 🎯 Root Cause

The statically configured NetworkManager profile contains the wrong DNS resolver address:

```text
Configured: 192.168.10.54
Intended:   192.168.10.53
```

The configured address has no responding resolver in this lab.

The client therefore cannot resolve `portal.demo.test`, even though the application server is reachable and the HTTPS service works.

# 🛠️ Resolution

## 1. Record the Original Setting

```bash
nmcli -g ipv4.dns connection show "Demo LAN"
```

Recorded value:

```text
192.168.10.54
```

The other profile settings are already correct.

## 2. Update the Managed Connection Profile

```bash
sudo nmcli connection modify "Demo LAN" ipv4.dns "192.168.10.53"
```

Reapply the profile settings to the active interface:

```bash
sudo nmcli device reapply eth0
```

This updates the persistent profile and requests that NetworkManager apply the change to the active device.

> [!important] Environment-specific application
> Reapply support depends on the setting and NetworkManager version. If the change cannot be reapplied, schedule a profile reactivation from a local console or with an acceptable interruption window. Do not blindly reconnect the interface during a remote session.

Do not edit `/etc/resolv.conf` directly when it is generated by NetworkManager.

# ✅ Verification

## 1. Confirm the Active DNS Configuration

```bash
nmcli -g IP4.DNS device show eth0
cat /etc/resolv.conf
```

Expected resolver:

```text
192.168.10.53
```

## 2. Test Normal DNS Resolution

Run without an explicit server override:

```bash
dig portal.demo.test. A +time=2 +tries=1
```

Expected results:

- Status: `NOERROR`.
- Answer: `192.168.10.50`.
- Responding server: `192.168.10.53`.

Check the system resolution path:

```bash
getent ahosts portal.demo.test
```

Expected result: entries containing `192.168.10.50`.

## 3. Test Normal Application Access

Run without `--resolve`:

```bash
curl --verbose --connect-timeout 5 --max-time 15 --output /dev/null https://portal.demo.test/
```

Expected outcomes:

- The hostname resolves.
- TCP and TLS succeed.
- The application returns HTTP `200`.

Finally, reopen the original URL in the browser and confirm that the portal loads.

## Verification Summary

| Test | Before Correction | After Correction |
|---|---|---|
| ICMP to application server | Succeeds | Succeeds |
| DNS through configured resolver | Times out | Returns expected address |
| DNS through intended resolver | Succeeds | Succeeds |
| HTTPS with DNS override | Succeeds | Succeeds |
| HTTPS through normal resolution | Fails | Succeeds |

These results describe the simulated scenario. A reproduced lab should record actual observations.

# 🛡️ Prevention

- Document intended DNS resolver addresses.
- Review static profiles before activating them.
- Verify active settings after configuration changes.
- Test internal name resolution as well as numeric-address reachability.
- Manage resolver settings through the component that owns them.
- Preserve relevant configuration and test results in troubleshooting records.

# 🔗 Related Notes

- [[DNS]]
- [[IP Addressing]]
- [[TCP and UDP]]
- [[ip]]
- [[ping]]
- [[dig]]
- [[Network Connectivity Troubleshooting]]
- [[network-diagnostics]]

# 📌 Notes

- The DNS resolver and client are on the same subnet, so the default gateway is not responsible for this failed query.
- A working service accessed with `--resolve` does not prove that normal name resolution works.
- Flushing a cache would not correct the wrong resolver address.
- To roll back the profile change, restore the recorded DNS value and reapply it. In this lab, that restores the original fault.
- If reproducing the case on Windows, use the same evidence sequence with [[ipconfig]] and [[Test-NetConnection]].

Reference: [NetworkManager — nmcli Documentation](https://networkmanager.dev/docs/api/latest/nmcli.html).