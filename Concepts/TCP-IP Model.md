---
Concept: TCP-IP Model
Description: "The four-layer model used to describe communication across IP networks and the Internet."
tags:
  - concept
  - networking
  - tcp-ip
  - fundamentals
---

# 🧠 Concept

The **TCP/IP model** describes how protocols work together to exchange data across interconnected networks.

Its name comes from **Transmission Control Protocol (TCP)** and **Internet Protocol (IP)**, but the protocol suite includes many other protocols, such as UDP, ICMP and DNS.

The model is commonly represented using **four layers**.

## 🧱 The Four Layers

| Layer | Name | Main Responsibility | Protocols and Technologies |
|---:|---|---|---|
| 4 | Application | Defines messages and communication behavior for application services | HTTP, DNS, DHCP, SSH, SMTP |
| 3 | Transport | Provides communication between application endpoints | TCP, UDP |
| 2 | Internet | Provides IP addressing and packet forwarding across networks | IPv4, IPv6, ICMP |
| 1 | Network Access | Transfers data over the local link and physical medium | Ethernet, Wi-Fi |

> [!note] Alternative representations
> Some educational models use five layers by separating Network Access into Data Link and Physical. This vault uses the four-layer representation consistently.

## 🔄 TCP/IP and OSI Mapping

The [[OSI Model]] separates communication into seven layers. TCP/IP groups the same broad functions into fewer layers.

| TCP/IP Layer | Corresponding OSI Layers |
|---|---|
| Application | Application, Presentation, Session |
| Transport | Transport |
| Internet | Network |
| Network Access | Data Link, Physical |

This is a conceptual comparison. Real protocols do not always fit neatly into a single layer.

For example, TLS provides encryption and data representation functions, but is usually implemented between an application protocol and its underlying transport.

## 📦 Encapsulation and Decapsulation

For a typical TCP/IP exchange over Ethernet:

1. The **Application layer** creates a message.
2. The **Transport layer** adds TCP or UDP information, including port numbers.
3. The **Internet layer** adds IP information, including source and destination addresses.
4. The **Network Access layer** places the IP packet inside a frame and transmits it over the local medium.

| Layer | Common Data Unit | Addressing or Identification |
|---|---|---|
| Application | Message or data | Application-specific names and identifiers |
| Transport | TCP segment or UDP datagram | Port numbers |
| Internet | IP packet | IP addresses |
| Network Access | Frame | Link-layer addresses, such as Ethernet MAC addresses |

The receiving host processes the data in reverse through **decapsulation**.

> [!note] Encapsulation varies
> TCP and UDP are common transport protocols, but not every IP packet contains one of them. ICMP messages, for example, are carried directly inside IP packets.

## 🌐 Example: An HTTPS Connection

Consider a browser accessing `https://example.com` using HTTP/1.1 or HTTP/2 over TCP.

| Stage | What Happens |
|---|---|
| Name resolution | The client resolves `example.com` to an IP address |
| Transport connection | The client establishes a TCP connection to destination port `443` |
| Secure communication | TLS negotiates the protected connection and validates the server certificate |
| Application exchange | The browser sends an HTTP request and receives a response |
| Network delivery | IP and the local link carry the traffic between endpoints |

The client's routing table determines whether the destination is directly reachable or requires a next-hop router.

For a remote destination on an Ethernet network:

- The **destination IP address** identifies the remote server.
- The **destination MAC address** in the initial frame identifies the local next hop, usually the default gateway.

Routers replace link-layer framing as packets move between links.

> [!note] HTTP/3
> HTTP/3 uses QUIC over UDP rather than TCP. The example above specifically describes HTTPS over TCP.

## 🛠️ Troubleshooting by Layer

| Layer | What to Investigate | Tools or Evidence |
|---|---|---|
| Network Access | Interface state, local link and neighbor resolution | Physical checks and [[ip]] |
| Internet | IP configuration, routing and IP reachability | [[ip]], [[ipconfig]], [[ping]] |
| Transport | Reachability of the required TCP port | [[Test-NetConnection]] |
| Application | DNS answers, service responses and application errors | [[dig]], application output and logs |

A problem observed at one layer may originate elsewhere.

For example, a DNS lookup may fail because:

- The configured resolver is incorrect.
- No route exists to the resolver.
- A firewall blocks the required traffic.
- The resolver cannot answer the query.

Follow [[Network Connectivity Troubleshooting]] to investigate these dependencies systematically.

## 🔑 Key Points

- TCP/IP is a protocol suite, not just TCP and IP.
- This vault uses the four-layer model.
- IP provides addressing and packet delivery across networks, without guaranteeing delivery.
- TCP provides a reliable, ordered byte stream.
- UDP carries datagrams without built-in delivery or ordering guarantees.
- Port numbers distinguish transport endpoints.
- Link-layer delivery and IP routing have different scopes.
- Successful connectivity at one layer does not prove that the application works.

## 🔗 Related Notes

- [[OSI Model]]
- [[IP Addressing]]
- [[TCP and UDP]]
- [[DNS]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

## 📌 Notes

Use TCP/IP to explain how actual protocol exchanges work, and OSI to discuss their functions in greater detail.

When troubleshooting, distinguish name resolution, IP reachability, transport connectivity and application behavior.

Reference: [RFC 1122 — Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122.html).