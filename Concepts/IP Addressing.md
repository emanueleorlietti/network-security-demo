---
Concept: IP Addressing
Description: "How IPv4 and IPv6 addresses identify network interfaces and support routing between networks."
tags:
  - concept
  - networking
  - ip
  - ipv4
  - ipv6
  - subnetting
---

# 🧠 Concept

**IP addressing** provides logical addresses for network interfaces and enables packets to be routed between networks.

An interface can have multiple IP addresses. An address must be unique within its relevant scope to avoid conflicts.

The two versions are:

- **IPv4** — Uses 32-bit addresses.
- **IPv6** — Uses 128-bit addresses.

IP operates at the Internet layer of the [[TCP-IP Model]], corresponding to Layer 3 of the [[OSI Model]].

## 🌐 IPv4 Addresses

An IPv4 address contains four decimal octets separated by dots:

```text
192.168.10.34
```

Each octet represents 8 bits and ranges from `0` to `255`.

An address is interpreted together with its **subnet mask** or **prefix length**, which separates:

- The **network portion** — Identifies the subnet.
- The **host portion** — Identifies an address within that subnet.

## 🎭 Subnet Masks and CIDR

**Classless Inter-Domain Routing (CIDR)** notation expresses the number of network bits after a slash.

```text
192.168.10.34/24
```

The `/24` prefix means that the first 24 bits identify the network. The remaining 8 bits provide 256 possible addresses.

Its equivalent subnet mask is:

```text
255.255.255.0
```

| Prefix | Subnet Mask | Total Addresses | Traditional Usable Host Addresses |
|---:|---|---:|---:|
| `/24` | `255.255.255.0` | 256 | 254 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/27` | `255.255.255.224` | 32 | 30 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/29` | `255.255.255.248` | 8 | 6 |
| `/30` | `255.255.255.252` | 4 | 2 |

For these conventional subnets, the first address identifies the network and the last is the directed broadcast address.

> [!note] Special prefixes
> A `/31` can provide two usable addresses on an IPv4 point-to-point link. A `/32` identifies one address and is commonly used for host routes. The usual “total addresses minus two” calculation does not apply to these prefixes.

## 🧮 Subnet Example

Consider this interface configuration:

```text
IP address: 192.168.10.34
Prefix:     /27
Mask:       255.255.255.224
```

A `/27` leaves 5 host bits:

```text
2^5 = 32 addresses
```

The address belongs to the block from `192.168.10.32` through `192.168.10.63`.

| Property | Value |
|---|---|
| Network address | `192.168.10.32` |
| First usable host | `192.168.10.33` |
| Last usable host | `192.168.10.62` |
| Directed broadcast address | `192.168.10.63` |
| Total addresses | 32 |
| Traditional usable host addresses | 30 |

With a gateway at `192.168.10.33`:

- `192.168.10.50` is inside the configured subnet.
- `192.168.10.70` is outside the configured subnet.

Under a typical configuration, the host delivers traffic to the first destination directly and sends traffic to the second through a router.

## 🧭 Routing and the Default Gateway

A host uses its **routing table** to decide where to send a packet.

The route with the **longest matching prefix** takes precedence. Other selection rules, such as metrics, distinguish otherwise equivalent routes.

A **default route** is used when no more specific route matches:

| IP Version | Default Route |
|---|---|
| IPv4 | `0.0.0.0/0` |
| IPv6 | `::/0` |

The **default gateway** is the next-hop router associated with that route.

> [!important] Local and remote delivery
> The subnet prefix helps establish directly connected routes, but the routing table determines the actual forwarding decision. Static routes, VPNs and policy routing can change the expected path.

## 🔒 Private IPv4 Addresses

RFC 1918 reserves these ranges for private networks:

| Range | CIDR |
|---|---|
| `10.0.0.0 – 10.255.255.255` | `10.0.0.0/8` |
| `172.16.0.0 – 172.31.255.255` | `172.16.0.0/12` |
| `192.168.0.0 – 192.168.255.255` | `192.168.0.0/16` |

These addresses are not globally routable on the public Internet. Networks commonly use NAT to provide outbound IPv4 access.

> [!note] Private does not mean secure
> Private addressing does not provide encryption or access control. Security still depends on the network configuration and appropriate protections.

## 📌 Special IPv4 Addresses

| Address or Range | Purpose |
|---|---|
| `127.0.0.0/8` | Loopback; commonly `127.0.0.1` |
| `169.254.0.0/16` | Link-local addressing |
| `0.0.0.0` | Unspecified address; also used by applications to bind to all local IPv4 interfaces |
| `0.0.0.0/0` | Default route |
| `255.255.255.255` | Limited broadcast on the local link |
| `192.0.2.0/24` | Documentation examples |
| `198.51.100.0/24` | Documentation examples |
| `203.0.113.0/24` | Documentation examples |

An unexpected `169.254.x.x` address on a client configured for automatic IPv4 addressing can indicate that it did not obtain a DHCP lease. Check the interface and DHCP configuration before drawing a conclusion.

## 🌍 IPv6 Addresses

An IPv6 address contains eight groups of hexadecimal digits:

```text
2001:db8:1234:5678:0000:0000:0000:0010
```

Leading zeros in each group can be omitted. One continuous sequence of zero groups can be replaced with `::`:

```text
2001:db8:1234:5678::10
```

The `::` compression can appear only once in an address.

| Address or Range | Purpose |
|---|---|
| `2000::/3` | Main global unicast allocation range |
| `fc00::/7` | Unique local addressing; locally generated prefixes use `fd00::/8` |
| `fe80::/10` | Link-local addressing |
| `ff00::/8` | Multicast |
| `::1/128` | Loopback |
| `::/128` | Unspecified address |
| `2001:db8::/32` | Documentation examples |

A `/64` is the standard prefix size for most IPv6 LANs using Stateless Address Autoconfiguration (SLAAC).

IPv6 has **no broadcast addresses**. It uses multicast for functions that require communication with groups of hosts.

> [!note] Link-local scope
> IPv6 link-local addresses are valid only on their local link. Tools may require an interface identifier, such as `fe80::1%eth0`, to distinguish the intended link.

## 🛠️ Addressing Checks During Troubleshooting

| Check | What to Confirm |
|---|---|
| Interface | The expected interface is active |
| IP address | The address belongs to the intended network |
| Prefix or mask | The subnet boundary is correct |
| Address conflicts | Another device is not using the same address |
| Routing table | A suitable route exists for the destination |
| Default gateway | The configured next hop is appropriate and reachable |
| DNS configuration | The client uses the intended resolvers |
| Address family | IPv4 and IPv6 are tested separately when relevant |

Use [[ip]] on Linux and [[ipconfig]] on Windows to inspect addressing information.

DNS configuration is separate from IP addressing: a host can have valid addressing and routing while name resolution fails.

## 🔑 Key Points

- IPv4 addresses contain 32 bits; IPv6 addresses contain 128 bits.
- Always interpret an address together with its prefix.
- The routing table determines the next hop.
- A default gateway handles destinations without a more specific matching route.
- Private IPv4 addresses are not globally routable.
- IPv6 uses multicast instead of broadcast.
- Incorrect addressing, prefixes or routes can cause connectivity failures.

## 🔗 Related Notes

- [[OSI Model]]
- [[TCP-IP Model]]
- [[TCP and UDP]]
- [[DNS]]
- [[ip]]
- [[ipconfig]]
- [[Network Connectivity Troubleshooting]]

## 📌 Notes

Use documentation ranges for fictional public addresses and private ranges for clearly identified lab networks. Never include real organizational addressing in a public troubleshooting example.

References:

- [RFC 1918 — Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918.html)
- [RFC 4632 — Classless Inter-domain Routing](https://www.rfc-editor.org/rfc/rfc4632.html)
- [RFC 4291 — IPv6 Addressing Architecture](https://www.rfc-editor.org/rfc/rfc4291.html)
- [RFC 5737 — IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737.html)
- [RFC 3849 — IPv6 Address Prefix Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc3849.html)