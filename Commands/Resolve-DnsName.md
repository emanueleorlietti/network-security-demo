---
Command: "Resolve-DnsName"
Description: "Query DNS records through the Windows DnsClient module."
OS: "Windows"
tags:
  - command
  - windows
  - networking
  - dns
  - troubleshooting
---

## Purpose and execution context

Use Windows PowerShell with the DnsClient module; check `Get-Command Resolve-DnsName`. Ordinary queries generally need no elevation.

## Examples

```powershell
Resolve-DnsName -Name example.com. -Type A -DnsOnly -NoHostsFile
Resolve-DnsName -Name portal.demo.test. -Type A -Server 192.168.10.53 -DnsOnly -NoHostsFile
Resolve-DnsName -Name portal.demo.test. -Type A -Server 192.168.10.53 -DnsOnly -NoHostsFile -TcpOnly
```

The private address/name require the fictional lab in [[Network Connectivity Troubleshooting]]. `-Type` selects the record, `-Server` the resolver, `-DnsOnly` DNS-only resolution, `-NoHostsFile` skips the hosts file, and `-TcpOnly` selects TCP.

## Interpretation and limitations

Inspect returned record types/addresses and errors. A successful explicit query does not validate the application's resolver path, its caches, or service connectivity. Compare only DNS views expected to match.

## Impact and validation

Active queries can populate caches and generate logs. Do not query internal names through an external resolver. Recheck normal application access without an explicit server override after a fix.

## Related notes

- [[DNS]]
- [[dig]]
- [[DNS Resolution Failure]]

## References and validation status

[Microsoft: Resolve-DnsName](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname). Reviewed 2026-10-17; examples not executed against live resolvers.
