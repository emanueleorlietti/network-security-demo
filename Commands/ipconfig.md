---
Command: "ipconfig"
Description: "Displays Windows IP configuration and provides DHCP lease and DNS cache operations."
OS: "Windows"
tags:
  - command
  - windows
  - networking
  - ip
  - dns
  - troubleshooting
---

## Execution context and impact

Windows, from PowerShell or Command Prompt. Inspection and state-changing examples are separated below. Before a lease operation, identify the exact adapter, preserve configuration and ensure recovery access; a successful renewal is not guaranteed and the previous lease cannot be assumed recoverable.

# 🛠️ Command

```powershell
ipconfig
```

The **ipconfig** command displays TCP/IP configuration for Windows network adapters.

It can be run from PowerShell or Command Prompt.

Without parameters, it shows basic information such as IP addresses, subnet masks and default gateways.

# 🔧 Common Options

| Option | Purpose |
|---|---|
| `/all` | Displays detailed configuration for all adapters |
| `/displaydns` | Displays the local DNS resolver cache |
| `/flushdns` | Clears the local DNS resolver cache |
| `/release "ADAPTER"` | Releases the DHCPv4 configuration for the selected adapter |
| `/renew "ADAPTER"` | Requests renewed DHCPv4 configuration for the selected adapter |
| `/release6 "ADAPTER"` | Releases DHCPv6 configuration for the selected adapter |
| `/renew6 "ADAPTER"` | Renews DHCPv6 configuration for the selected adapter |
| `/?` | Displays built-in help |

> [!important] Inspection and changes
> `/all` and `/displaydns` inspect state. Flushing the DNS cache or releasing and renewing DHCP configuration changes local state. Save relevant diagnostic evidence before making changes.

# 📚 Examples

## Display Basic Configuration

```powershell
ipconfig
```

Check the active adapter's:

- IPv4 and IPv6 addresses.
- IPv4 subnet mask.
- Default gateway.
- Connection status.

Systems may also show disconnected, virtual or VPN adapters. Identify the adapter relevant to the failing connection.

## Display Detailed Configuration

```powershell
ipconfig /all
```

Illustrative excerpt:

```text
Ethernet adapter Ethernet:

   DHCP Enabled. . . . . . . . . . . : Yes
   IPv4 Address. . . . . . . . . . . : 192.168.10.34(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.224
   Default Gateway . . . . . . . . . : 192.168.10.33
   DHCP Server . . . . . . . . . . . : 192.168.10.33
   DNS Servers . . . . . . . . . . . : 192.168.10.53
```

| Field | What to Verify |
|---|---|
| `DHCP Enabled` | Whether dynamic configuration is expected |
| `IPv4 Address` | Whether the address belongs to the intended network |
| `Subnet Mask` | Whether the subnet boundary is correct |
| `Default Gateway` | Whether the expected next-hop router is configured |
| `DHCP Server` | Which server supplied the DHCP configuration |
| `DNS Servers` | Whether the intended resolvers are configured |

In this example, `255.255.255.224` corresponds to `/27`, placing the client in `192.168.10.32/27`.

The output is illustrative. Adapter names, labels and formatting vary with Windows configuration and display language.

## Inspect the DNS Resolver Cache

```powershell
ipconfig /displaydns
```

This displays cached DNS information, including positive and negative entries where present.

It can help explain why a client continues using an earlier answer after a DNS record changes.

> [!note] Cache scope
> This command shows the Windows DNS Client resolver cache. Applications may maintain separate caches or use their own resolvers.

## Clear the DNS Resolver Cache

After recording relevant cached information:

```powershell
ipconfig /flushdns
```

Run from an elevated terminal if required by the environment.

Clearing the cache causes subsequent lookups to obtain fresh information through the configured resolution path.

It does not:

- Change the configured DNS servers.
- Correct an authoritative DNS record.
- Clear a recursive resolver's cache.
- Clear every application's private cache.

## Renew DHCPv4 Configuration

For a DHCP-enabled adapter named `Ethernet`:

```powershell
ipconfig /renew "Ethernet"
```

This requests renewal of the adapter's DHCP configuration.

Use it when investigating lease-related issues or retrieving updated DHCP settings. It does not apply to a statically configured IPv4 address.

## Release and Renew DHCPv4 Configuration

When a full release is justified:

```powershell
ipconfig /release "Ethernet"
ipconfig /renew "Ethernet"
```

Replace `Ethernet` with the actual adapter name.

> [!warning] Connectivity interruption
> Releasing DHCP configuration can disconnect the adapter and interrupt a remote session. Renewal can fail, leaving the adapter without its previous configuration. Use this sequence only when the interruption is acceptable.

Avoid omitting the adapter name unless you intend to affect all applicable adapters.

## Renew DHCPv6 Configuration

```powershell
ipconfig /renew6 "Ethernet"
```

This concerns DHCPv6 configuration. IPv6 addresses and default routes can also come from SLAAC and Router Advertisements, so DHCPv6 renewal does not refresh every part of IPv6 configuration.

## Display Built-in Help

```powershell
ipconfig /?
```

Use the installed help to confirm syntax and supported options.

# 💡 Notes

- Most configuration inspection works without elevation.
- Some operations require administrator privileges.
- An unexpected `169.254.x.x` IPv4 address can indicate that a DHCP-enabled adapter did not obtain a lease.
- Multiple adapters can have different DNS settings and gateways.
- `ipconfig` does not display the full routing table or establish which route a connection uses.
- A configured gateway does not prove that it is reachable.
- Correct IP configuration does not prove that DNS or an application works.
- Record the initial configuration before releasing leases or clearing cached information.

# 🔗 Related Notes

- [[IP Addressing]]
- [[DNS]]
- [[TCP-IP Model]]
- [[Test-NetConnection]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

Reference: [Microsoft Learn — ipconfig](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig).
