---
Command: "ip"
Description: "Inspects and manages Linux network interfaces, IP addresses, routes and neighbor entries."
OS: "Linux"
tags:
  - command
  - linux
  - networking
  - ip
  - troubleshooting
---

## Execution context and impact

Linux with iproute2; inspect the current network namespace and relevant interface. The examples below inspect state. The syntax summary is notation, not a runnable test. Stop `ip monitor` with Ctrl+C; no network configuration rollback is needed for these inspection examples.

# 🛠️ Command

```bash
ip [OPTIONS] OBJECT COMMAND
```

The **ip** command is part of the `iproute2` suite. It provides access to Linux network configuration and state.

Common objects include:

| Object | Purpose |
|---|---|
| `link` | Network interfaces and link properties |
| `address` | IPv4 and IPv6 addresses |
| `route` | Routing tables and route selection |
| `neigh` | IPv4 ARP and IPv6 neighbor entries |
| `rule` | Policy routing rules |

The examples below inspect existing state without changing the configuration.

# 🔧 Common Options

| Option | Purpose |
|---|---|
| `-brief` | Displays compact output for supported objects |
| `-4` | Restricts output to IPv4 |
| `-6` | Restricts output to IPv6 |
| `-details` | Displays additional information |
| `-statistics` | Displays statistics where supported |
| `-json` | Produces JSON output |
| `-pretty` | Formats JSON output for readability |

## Compact Address Overview

```bash
ip -brief address show
```

## IPv4 or IPv6 Addresses

```bash
ip -4 address show
ip -6 address show
```

## Interface Statistics

```bash
ip -statistics link show
```

## JSON Output

```bash
ip -json -pretty address show
```

# 📚 Examples

## Inspect Interfaces and Addresses

```bash
ip -brief address show
```

Illustrative output:

```text
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             192.168.10.34/27
```

This shows interface names, operational state and assigned addresses.

The prefix `/27` must be interpreted together with the address. See [[IP Addressing]].

> [!note] Interface names
> Replace `eth0` with the interface on your system, such as `enp1s0`, `ens18` or `wlan0`. The loopback interface reporting `UNKNOWN` is normally expected.

## Inspect Link State

```bash
ip link show dev eth0
```

Look for:

- `UP` — The interface is administratively enabled.
- `LOWER_UP` — The driver reports the lower-layer link as available.
- `state` — The reported operational state.
- `mtu` — The interface's maximum transmission unit.
- `link/ether` — The Ethernet MAC address, where applicable.

Administrative state and operational state describe different conditions. An enabled interface can still lack a working physical connection.

## Inspect Addresses on One Interface

```bash
ip address show dev eth0
```

Check the address, prefix length, scope and any relevant flags.

An interface may have multiple addresses, including both IPv4 and IPv6.

## Inspect the IPv4 Routing Table

```bash
ip -4 route show
```

Illustrative output:

```text
default via 192.168.10.33 dev eth0
192.168.10.32/27 dev eth0 proto kernel scope link src 192.168.10.34
```

| Entry | Interpretation |
|---|---|
| `default via 192.168.10.33` | Traffic without a more specific matching route uses this gateway |
| `192.168.10.32/27 dev eth0` | This subnet is directly connected through `eth0` |
| `src 192.168.10.34` | Preferred source address for this route |

## Inspect the Selected Route to a Destination

```bash
ip route get 1.1.1.1
```

Illustrative output:

```text
1.1.1.1 via 192.168.10.33 dev eth0 src 192.168.10.34 uid 1000
    cache
```

The result identifies the selected next hop, outgoing interface and source address for the supplied lookup conditions.

> [!important] Route selection is not a connectivity test
> `ip route get` performs a route lookup. It does not send a probe or prove that the destination is reachable.

## Inspect IPv6 Routes

```bash
ip -6 route show
```

A default route may use a link-local gateway:

```text
default via fe80::1 dev eth0 proto ra metric 100
```

The interface is essential because a link-local address is scoped to a particular link.

## Inspect Neighbor Entries

```bash
ip neigh show dev eth0
```

Illustrative output:

```text
192.168.10.33 lladdr 02:00:00:00:00:33 REACHABLE
```

Neighbor entries associate local next-hop IP addresses with link-layer addresses.

| State | Interpretation |
|---|---|
| `REACHABLE` | Reachability was confirmed recently |
| `STALE` | The entry remains usable, but reachability has not been confirmed recently |
| `INCOMPLETE` | Address resolution is in progress |
| `FAILED` | Neighbor resolution failed |

A `STALE` entry is normal and does not by itself indicate a fault. A `FAILED` entry warrants checking the next hop, link and local network configuration.

## Inspect Policy Routing

```bash
ip rule show
ip route show table all
```

These checks are useful when VPNs, multiple interfaces or custom routing policies make the main routing table insufficient.

## Watch Network Changes

```bash
ip monitor link address route
```

This displays interface, address and route events as they occur. Press `Ctrl+C` to stop.

It can help identify changes during a connection loss or VPN reconnection.

# 💡 Notes

- Most inspection commands work without elevated privileges.
- Configuration changes generally require root or the relevant capabilities.
- Output depends on the kernel, `iproute2` version and network configuration.
- Commands inspect the current network namespace, which may differ inside a container.
- `ip` does not display DNS resolver configuration.
- A valid route does not prove that traffic passes through the path.
- For a remote destination, neighbor information normally concerns the next-hop router rather than the remote host.
- All output shown here is illustrative.

# 🔗 Related Notes

- [[IP Addressing]]
- [[OSI Model]]
- [[TCP-IP Model]]
- [[ping]]
- [[Network Connectivity Troubleshooting]]
- [[network-diagnostics]]

Reference: Run `man ip`, `man ip-address`, `man ip-link`, `man ip-route` and `man ip-neighbour` for the documentation installed on your system.
