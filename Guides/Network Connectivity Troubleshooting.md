---
Guide: "Network Connectivity Troubleshooting"
Description: "A structured procedure for separating local network, routing, DNS, transport and application failures."
Difficulty: "Intermediate"
tags:
  - guide
  - networking
  - dns
  - troubleshooting
---

# 🎯 Objective

Investigate a connectivity problem by testing its dependencies separately:

1. Local interface and IP configuration.
2. Routing and next-hop connectivity.
3. Name resolution.
4. Transport connectivity.
5. Application behavior.

The goal is to collect evidence before changing configuration and identify the smallest justified corrective action.

# 📋 Prerequisites

- Access to the affected Linux or Windows client.
- The destination hostname and expected service port.
- Knowledge of the intended network, gateway and DNS resolver.
- Permission to test the destination.
- Linux tools: `ip`, `ping`, `dig` and `curl`.
- Windows tools: `ipconfig`, `Test-NetConnection`, `Resolve-DnsName` and `curl.exe`.

See [[Resolve-DnsName]], [[curl]] and [[ss]] for additional tool context. All lab results below are expected outcomes, not measurements collected from your systems.

## Example Lab

Examples use this fictional environment:

| Component | Value |
|---|---|
| Client IPv4 address | `192.168.10.34/27` |
| Default gateway | `192.168.10.33` |
| Intended DNS resolver | `192.168.10.53` |
| Application hostname | `portal.demo.test` |
| Application IPv4 address | `192.168.10.50` |
| Application service | HTTPS over TCP port `443` |

The client, gateway, resolver and server belong to `192.168.10.32/27`.

The `.test` namespace is reserved for testing. These names and addresses require a configured lab; they are not working public endpoints.

Replace them with values appropriate to your environment.

# 🛠️ Procedure

## 1. Define the Failure

Record:

- The affected application and exact error.
- The hostname, protocol and port.
- When the problem began.
- Whether one client or several clients are affected.
- Whether the failure is constant or intermittent.
- Recent network, VPN or application changes.

Distinguish a name-resolution error from a connection timeout, certificate error or HTTP error.

If possible, compare with a working client on the same intended network.

> [!note] Keep comparisons relevant
> Clients on different VPNs, networks or DNS views may legitimately receive different answers and use different paths.

## 2. Inspect Local Configuration

### Linux

```bash
ip -brief link show
ip -brief address show
ip -4 route show
ip -6 route show
```

Inspect DNS configuration:

```bash
cat /etc/resolv.conf
```

On systems using `systemd-resolved`:

```bash
resolvectl status
```

`/etc/resolv.conf` may identify a local stub rather than the upstream DNS servers. Check per-interface resolver information when available.

### Windows

```powershell
ipconfig /all
```

Inspect routes if needed:

```powershell
Get-NetRoute -AddressFamily IPv4
Get-NetRoute -AddressFamily IPv6
```

Confirm:

- The relevant interface is active.
- Its address and prefix match the intended network.
- A suitable route exists.
- The expected DNS resolvers are configured.
- VPN or virtual adapters have not changed the intended path.

An unexpected IPv4 link-local address in `169.254.0.0/16` can indicate a DHCP problem on a client expecting a lease.

## 3. Check Route Selection and the Next Hop

### Linux

Inspect the selected route to the lab server:

```bash
ip route get 192.168.10.50
```

Because the server is in the client's subnet, normal delivery is direct rather than through the default gateway.

Test the gateway and inspect neighbor state:

```bash
ping -4 -n -c 4 -W 2 192.168.10.33
ip neigh show
```

For a remote destination, perform a separate route lookup:

```bash
ip route get 1.1.1.1
```

### Windows

```powershell
Test-NetConnection -ComputerName 192.168.10.50 -DiagnoseRouting -InformationLevel Detailed
Test-NetConnection -ComputerName 192.168.10.33
```

Use the selected route and source address to confirm that Windows is considering the expected interface.

> [!important] Interpret reachability carefully
> A route lookup does not send a connectivity probe. A failed ICMP test does not establish that a destination is unavailable. A successful gateway test establishes only that this local ICMP exchange worked.

If local delivery appears broken, investigate link state, subnet configuration, neighbor resolution and relevant network policies before proceeding.

## 4. Test DNS Separately

### Linux

Query through the default DNS path used by `dig`:

```bash
dig portal.demo.test. A +time=2 +tries=1
dig portal.demo.test. AAAA +time=2 +tries=1
```

Query the intended resolver explicitly:

```bash
dig @192.168.10.53 portal.demo.test. A +time=2 +tries=1
```

### Windows

```powershell
Resolve-DnsName -Name portal.demo.test. -Type A -DnsOnly -NoHostsFile
Resolve-DnsName -Name portal.demo.test. -Type AAAA -DnsOnly -NoHostsFile
```

Query the intended resolver explicitly:

```powershell
Resolve-DnsName -Name portal.demo.test. -Type A -Server 192.168.10.53 -DnsOnly -NoHostsFile
```

Compare:

- The responding resolver.
- Response status or reported error.
- Returned addresses and aliases.
- Whether results match the intended DNS view.

An absent `AAAA` record can be expected for an IPv4-only lab service.

### Compare DNS Transports When Relevant

Linux:

```bash
dig @192.168.10.53 portal.demo.test. A +time=2 +tries=1
dig @192.168.10.53 portal.demo.test. A +tcp +time=2
```

Windows:

```powershell
Resolve-DnsName -Name portal.demo.test. -Type A -Server 192.168.10.53 -DnsOnly -NoHostsFile -TcpOnly
```

Differences between UDP and TCP can indicate filtering or transport-specific problems.

Do not send internal names to public resolvers unless that is explicitly appropriate for the investigation.

## 5. Test the Service's Transport

### Linux

Use `curl` for the HTTPS service in the next step. Inspect its verbose output to distinguish connection, TLS and HTTP failures.

### Windows

Test the expected IP address:

```powershell
Test-NetConnection -ComputerName 192.168.10.50 -Port 443 -InformationLevel Detailed
```

Then test the hostname:

```powershell
Test-NetConnection -ComputerName portal.demo.test -Port 443 -InformationLevel Detailed
```

Compare `RemoteAddress`, `SourceAddress`, `InterfaceAlias` and `TcpTestSucceeded`.

If the address test succeeds and the hostname test fails, investigate resolution and address selection. Confirm that both tests target the same endpoint.

For a UDP service, use a protocol-aware query. A TCP test of the same numeric port does not test UDP.

## 6. Test the Application

### Linux

```bash
curl --verbose --connect-timeout 5 --max-time 15 --output /dev/null https://portal.demo.test/
```

### Windows

```powershell
curl.exe --verbose --connect-timeout 5 --max-time 15 --output NUL https://portal.demo.test/
```

Inspect the stages in order:

1. Hostname resolution.
2. Connection establishment.
3. TLS negotiation and certificate validation.
4. HTTP response.

An HTTP error response proves that an HTTP exchange occurred, but may indicate authentication, authorization or application failure.

### Bypass DNS While Preserving the Hostname

Linux:

```bash
curl --verbose --connect-timeout 5 --max-time 15 --noproxy portal.demo.test --resolve portal.demo.test:443:192.168.10.50 --output /dev/null https://portal.demo.test/
```

Windows:

```powershell
curl.exe --verbose --connect-timeout 5 --max-time 15 --noproxy portal.demo.test --resolve portal.demo.test:443:192.168.10.50 --output NUL https://portal.demo.test/
```

This connects directly to the specified address while preserving the URL hostname for TLS and HTTP.

Use this direct-path test only where direct access is permitted. It bypasses a configured proxy for this hostname and therefore may differ from the application's normal path.

> [!important] Keep certificate validation enabled
> Do not add `-k` merely to obtain a successful result. A lab HTTPS service needs a certificate trusted by the client and valid for `portal.demo.test`.

## 7. Identify and Apply the Correction

Use the collected evidence to choose a targeted action.

| Evidence | Candidate Area |
|---|---|
| Interface lacks a working link | Cable, wireless association or interface state |
| Address or prefix differs from the intended configuration | Static settings or DHCP configuration |
| Selected route uses an unexpected path | Routing, VPN or policy configuration |
| Intended DNS resolver answers, configured resolver fails | Client DNS configuration or resolver availability |
| DNS returns an incorrect record | Authoritative zone data or cached information |
| Correct address resolves, TCP connection fails | Listener, filtering, routing or return path |
| TCP connects, TLS fails | Certificate, trust, hostname or TLS configuration |
| TLS succeeds, HTTP returns an error | Application, authentication or authorization |

These are investigation areas, not automatic diagnoses.

Record the original setting, make one justified change and repeat the relevant test.

# ✅ Verification

After the correction:

1. Recheck the affected configuration.
2. Repeat the test that originally failed.
3. Test normal hostname resolution without an explicit resolver override.
4. Confirm the expected transport and application response.
5. Verify the original user workflow.

For this lab, ordinary access should succeed without `--resolve`:

Linux:

```bash
curl --verbose --connect-timeout 5 --max-time 15 --output /dev/null https://portal.demo.test/
```

Windows:

```powershell
Test-NetConnection -ComputerName portal.demo.test -Port 443
curl.exe --verbose --connect-timeout 5 --max-time 15 --output NUL https://portal.demo.test/
```

Record actual results and the verification time. Do not treat simulated output as observed evidence.

# ↩️ Rollback

The inspection steps do not intentionally modify network configuration. Queries and connection attempts can populate caches and generate logs.

For any corrective change:

- Restore the recorded original settings if the change causes a regression.
- Revert changes through the component that manages them, such as NetworkManager, Windows adapter settings or DHCP.
- Confirm that connectivity returns to the pre-change state.

DNS cache flushing cannot restore the previous cache contents. Capture relevant entries before clearing them.

# ⚠️ Troubleshooting

- **Problem:** Ping fails, but the application works.  
  **Solution:** Investigate ICMP policy. Do not use ping alone to declare an outage.

- **Problem:** An explicit DNS query succeeds, but the application still fails.  
  **Solution:** Inspect application caches, hosts files, encrypted DNS, VPN settings and proxy behavior.

- **Problem:** One address works, but a hostname test fails.  
  **Solution:** Compare the actual selected addresses, including IPv4 and IPv6.

- **Problem:** `curl --resolve` fails with a certificate error.  
  **Solution:** Verify the certificate name and trust chain. DNS bypass does not bypass TLS validation.

- **Problem:** Route diagnostics look correct, but traffic fails.  
  **Solution:** Investigate forwarding, filtering, the return path and destination behavior.

# 🔗 Related Notes

- [[OSI Model]]
- [[TCP-IP Model]]
- [[IP Addressing]]
- [[TCP and UDP]]
- [[DNS]]
- [[ip]]
- [[ping]]
- [[dig]]
- [[ipconfig]]
- [[Test-NetConnection]]
- [[network-diagnostics]]
- [[DNS Resolution Failure]]

# 📚 References

- Local documentation: `man ip`, `man ping`, `man dig`, `man curl`
- [Microsoft Learn — Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection)
- [Microsoft Learn — Resolve-DnsName](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname)
- [curl — Command-line Documentation](https://curl.se/docs/manpage.html)
- [RFC 2606 — Reserved Top Level DNS Names](https://www.rfc-editor.org/rfc/rfc2606.html)
