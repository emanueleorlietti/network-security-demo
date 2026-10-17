---
Command: "ping"
Description: "Tests IP reachability and measures round-trip time using ICMP Echo requests on Linux."
OS: "Linux"
tags:
  - command
  - linux
  - networking
  - icmp
  - troubleshooting
---

## Execution context and impact

Linux iputils syntax; confirm installed options with `ping -h`. These are active ICMP probes to authorized targets. Use the count/deadline examples for bounded tests; results depend on policy and address-family selection.

# 🛠️ Command

```bash
ping [OPTIONS] DESTINATION
```

The **ping** command sends ICMP Echo requests and reports the replies it receives.

It helps assess:

- IP reachability.
- Round-trip time.
- Loss of probe responses during the test.

These examples use the Linux **iputils** implementation. Options can differ on other operating systems or implementations.

# 🔧 Common Options

| Option | Purpose |
|---|---|
| `-c COUNT` | Limits the number of requests |
| `-4` | Uses IPv4 |
| `-6` | Uses IPv6 |
| `-n` | Disables reverse DNS lookups in output |
| `-i INTERVAL` | Sets the interval between requests in seconds |
| `-W TIMEOUT` | Sets the response timeout in seconds |
| `-w DEADLINE` | Sets an overall deadline in seconds |
| `-I INTERFACE` | Selects a source interface or source address |
| `-s SIZE` | Sets the ICMP payload size in bytes |

## Send Four Requests

```bash
ping -c 4 1.1.1.1
```

## Use IPv4 Without Reverse Lookups

```bash
ping -4 -n -c 4 1.1.1.1
```

## Bound the Test Duration

```bash
ping -n -c 4 -W 2 -w 10 1.1.1.1
```

The response timeout and overall deadline serve different purposes. A deadline can end the test before all requested probes complete.

# 📚 Examples

## Test the Local IPv4 Stack

```bash
ping -4 -n -c 4 127.0.0.1
```

A successful loopback test confirms that local ICMP communication works. It does not test the physical interface, local network or Internet connection.

## Test the Default Gateway

First identify the gateway:

```bash
ip -4 route show default
```

Then test its address:

```bash
ping -4 -n -c 4 192.168.10.33
```

This example assumes that `192.168.10.33` is the configured gateway. Replace it with the value from your environment.

If the test fails, inspect interface state, addressing, neighbor information and ICMP policy.

## Test an External IP Address

```bash
ping -4 -n -c 4 1.1.1.1
```

Using a numeric address avoids a forward DNS lookup for the destination.

Replies confirm that ICMP traffic reached the destination and returned during the test. They do not establish that a particular application or TCP port is available.

## Test a Hostname

```bash
ping -4 -n -c 4 example.com
```

The system must resolve the hostname before sending probes.

If resolution fails, no ICMP reachability test occurs. Use [[dig]] to investigate DNS separately.

> [!note] The scope of `-n`
> `-n` disables reverse lookups when formatting output. It does not prevent the forward lookup required when the destination is a hostname.

## Interpret the Output

Illustrative output:

```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=57 time=12.4 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=57 time=12.8 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=57 time=12.1 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=57 time=12.5 ms

--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 12.100/12.450/12.800/0.250 ms
```

| Field | Meaning |
|---|---|
| `icmp_seq` | Sequence number identifying the probe |
| `ttl` | Remaining IPv4 Time to Live in the received reply |
| `time` | Round-trip time for that probe |
| `packet loss` | Percentage of sent probes without a received reply |
| `min/avg/max/mdev` | Minimum, average, maximum and variation of round-trip times |

The reply's TTL does not reveal an exact hop count unless its initial value is known.

## Select an Interface

```bash
ping -4 -n -c 4 -I eth0 1.1.1.1
```

Replace `eth0` with the intended interface.

This can help investigate hosts with multiple interfaces or VPN connections. Routing and policy rules still affect delivery.

## Test IPv6

```bash
ping -6 -n -c 4 ::1
```

For a link-local next hop, identify the outgoing interface:

```bash
ping -6 -n -c 4 -I eth0 fe80::1
```

Use the actual gateway address and interface from your environment.

IPv4 and IPv6 can fail independently. A successful test of one does not prove that the other works.

# 💡 Notes

- A failed ping does not prove that the destination is unavailable. ICMP may be filtered or rate-limited.
- A successful ping does not prove that DNS, TLS or an application works.
- Ping does not test TCP or UDP ports.
- Probe loss can result from either the forward path, return path or destination behavior.
- Small samples provide limited evidence about intermittent loss.
- Routers may forward traffic while declining to answer Echo requests themselves.
- Errors such as `Network is unreachable` suggest a local routing issue; other errors require interpretation in context.
- Permissions depend on system configuration, socket support and executable capabilities.
- All output shown here is illustrative.

# 🔗 Related Notes

- [[IP Addressing]]
- [[OSI Model]]
- [[TCP-IP Model]]
- [[ip]]
- [[dig]]
- [[Test-NetConnection]]
- [[Network Connectivity Troubleshooting]]
- [[network-diagnostics]]

Reference: Run `man ping` for the documentation installed on your system.
