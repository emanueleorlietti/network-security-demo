---
Command: "Test-NetConnection"
Description: "Tests Windows network connectivity through ICMP probes, TCP connection attempts and route diagnostics."
OS: "Windows"
tags:
  - command
  - windows
  - networking
  - tcp
  - troubleshooting
---

## Execution context and impact

Windows with the NetTCPIP module. Check availability using `Get-Command Test-NetConnection`. Tests generate network traffic; do not run repeated probes without a bound. Parameter sets and selected destination address determine what is actually tested.

# 🛠️ Command

```powershell
Test-NetConnection -ComputerName DESTINATION
```

**Test-NetConnection** is a Windows PowerShell cmdlet from the `NetTCPIP` module.

Its main diagnostic modes are:

- ICMP reachability testing.
- TCP connection testing.
- Route tracing.
- Local route and source-address selection diagnostics.

Always specify the destination and choose the mode relevant to the investigation.

# 🔧 Common Options

| Parameter | Purpose |
|---|---|
| `-ComputerName` | Specifies a hostname or IP address |
| `-Port` | Tests a specific TCP destination port |
| `-InformationLevel Detailed` | Displays additional diagnostic information |
| `-InformationLevel Quiet` | Returns a Boolean result for the selected connectivity test |
| `-TraceRoute` | Traces the path using ICMP probes |
| `-Hops` | Limits the number of hops for route tracing |
| `-DiagnoseRouting` | Investigates local route and source-address selection |
| `-ConstrainInterface` | Applies an interface constraint to route diagnostics |
| `-ConstrainSourceAddress` | Applies a source-address constraint to route diagnostics |

These parameters belong to different parameter sets. They cannot all be combined in one command.

# 📚 Examples

## Test ICMP Reachability

```powershell
Test-NetConnection -ComputerName 1.1.1.1
```

Inspect `PingSucceeded`.

This mode does not test a TCP port. A failed ICMP test can result from filtering even when an application is reachable.

## Test a TCP Port

```powershell
Test-NetConnection -ComputerName example.com -Port 443
```

Inspect `TcpTestSucceeded`.

A successful result confirms that a TCP connection was established to the selected address and port during the test.

It does not validate the TLS certificate, HTTP response or application functionality.

## Inspect Detailed TCP Results

The following example assumes a lab server at `192.168.10.50` with a TCP listener on port `443`:

```powershell
Test-NetConnection -ComputerName 192.168.10.50 -Port 443 -InformationLevel Detailed
```

Illustrative output excerpt:

```text
ComputerName     : 192.168.10.50
RemoteAddress    : 192.168.10.50
RemotePort       : 443
InterfaceAlias   : Ethernet
SourceAddress    : 192.168.10.34
TcpTestSucceeded : True
```

| Field | Interpretation |
|---|---|
| `RemoteAddress` | The destination address actually tested |
| `RemotePort` | The TCP destination port |
| `InterfaceAlias` | The selected outgoing interface |
| `SourceAddress` | The local source address |
| `TcpTestSucceeded` | Whether the TCP connection attempt succeeded |

Output fields vary by mode and environment. The excerpt above is simulated.

## Compare a Hostname and Its Expected Address

In a configured lab:

```powershell
Test-NetConnection -ComputerName portal.demo.test -Port 443
Test-NetConnection -ComputerName 192.168.10.50 -Port 443
```

If the address test succeeds but the hostname test fails, inspect name resolution and confirm which address the hostname test selected.

> [!important] Comparing equivalent destinations
> A hostname may resolve to several IPv4 or IPv6 addresses. Compare `RemoteAddress` before concluding that DNS is the cause.

The numeric-address test bypasses hostname resolution for the destination. It still does not reproduce a browser's TLS or HTTP behavior.

## Return a Boolean TCP Result

```powershell
$tcpReachable = Test-NetConnection -ComputerName 192.168.10.50 -Port 443 -InformationLevel Quiet

if ($tcpReachable) {
    "TCP connection succeeded."
} else {
    "TCP connection failed. Inspect detailed results."
}
```

Without `-Port`, Quiet mode reports the ICMP test result instead.

## Trace the Network Path

```powershell
Test-NetConnection -ComputerName 1.1.1.1 -TraceRoute -Hops 20
```

Intermediate devices may decline or filter probe responses.

A missing hop response does not prove that the device stopped forwarding traffic. This trace also does not reproduce the behavior of a TCP connection to a specific service.

## Diagnose Local Route Selection

```powershell
Test-NetConnection -ComputerName 1.1.1.1 -DiagnoseRouting -InformationLevel Detailed
```

Use this to inspect how Windows selects a route and source address.

It investigates local forwarding decisions rather than establishing application connectivity.

## Apply an Interface Constraint to Route Diagnostics

```powershell
Test-NetConnection -ComputerName 1.1.1.1 -DiagnoseRouting -ConstrainInterface 12 -InformationLevel Detailed
```

Replace `12` with the relevant interface index.

> [!note] Constraint scope
> `-ConstrainInterface` and `-ConstrainSourceAddress` apply to route diagnostics. They do not bind a `-Port` TCP connection test to a chosen interface or source address.

# 💡 Notes

- The cmdlet belongs to the Windows `NetTCPIP` module; it is not a cross-platform PowerShell networking command.
- TCP and ICMP results describe different tests and can legitimately differ.
- A failed TCP test alone does not identify whether the cause is routing, filtering, a missing listener or another condition.
- The cmdlet does not test UDP ports.
- Testing TCP port `53` does not prove that DNS queries succeed over either TCP or UDP.
- VPNs, multiple interfaces and address-family selection can affect the tested path.
- Specify the destination explicitly for reproducible documentation.

# 🔗 Related Notes

- [[TCP and UDP]]
- [[IP Addressing]]
- [[DNS]]
- [[ipconfig]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

Reference: [Microsoft Learn — Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection).
