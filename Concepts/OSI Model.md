---
Concept: OSI Model
Description: "A seven-layer reference model for understanding network communication and organizing troubleshooting."
tags:
  - concept
  - networking
  - osi
  - fundamentals
  - troubleshooting
---

# 🧠 Concept

The **Open Systems Interconnection (OSI) model** describes network communication through seven layers, each with a distinct responsibility.

It provides a shared vocabulary for explaining how systems communicate and identifying where a problem may occur.

Modern IP networks are better described by the [[TCP-IP Model]], but OSI remains useful for learning, documentation and troubleshooting.

## 🧱 The Seven Layers

| Layer | Name | Main Responsibility | Practical Examples |
|---:|---|---|---|
| 7 | Application | Provides network communication services to applications | HTTP, DNS, SSH |
| 6 | Presentation | Represents and transforms exchanged data | Encoding, serialization, compression, encryption |
| 5 | Session | Organizes and synchronizes exchanges between applications | Dialog control, synchronization points |
| 4 | Transport | Provides end-to-end data transfer and port addressing | TCP, UDP |
| 3 | Network | Provides logical addressing and routing between networks | IPv4, IPv6, ICMP |
| 2 | Data Link | Transfers frames over a local link | Ethernet frames, MAC addresses, VLAN tags |
| 1 | Physical | Transmits signals through a physical medium | Copper, fiber, radio signals |

> [!note] Real-world protocol mapping
> These examples are practical mappings. Internet protocols do not always fit a single OSI layer. TLS provides encryption associated with presentation functions, but is not an implementation of the OSI Presentation layer. Ethernet and Wi-Fi include both physical and data-link functions.

## 📦 Encapsulation and Decapsulation

For a typical TCP/IP exchange over Ethernet, the sending host wraps application data in successive protocol headers. Ethernet also adds a trailer.

| Stage | Common Data Unit | Information Added |
|---|---|---|
| Application processing | Data | Application-specific messages |
| Transport | TCP segment or UDP datagram | Source and destination ports |
| Network | IP packet | Source and destination IP addresses |
| Data Link | Ethernet frame | Source and destination MAC addresses, frame check sequence |
| Physical | Bits carried as signals | Physical encoding for the medium |

The receiving host processes these structures in reverse through **decapsulation**.

### Communication Across a Router

When forwarding an IP packet, a router:

1. Removes the incoming link-layer framing.
2. Examines the destination IP address and selects the next hop.
3. Updates relevant IP fields, such as TTL or Hop Limit.
4. Creates new link-layer framing for the outgoing interface.

For a remote destination, the host normally sends the initial Ethernet frame to its default gateway's MAC address, while the IP packet contains the remote destination's IP address.

## 🛠️ Troubleshooting with the OSI Model

Use the layers to organize evidence and narrow the investigation.

| Focus | Questions to Ask | Practical Checks |
|---|---|---|
| Physical | Is the interface connected? Is the link active? | Check cables, wireless signal and interface state |
| Data Link | Is local delivery working? Can the next hop be resolved? | Inspect neighbor information with [[ip]] |
| Network | Are the IP address, subnet and route correct? | Use [[ip]], [[ipconfig]] and [[ping]] |
| Transport | Can a connection reach the required TCP port? | Use [[Test-NetConnection]] |
| Upper Layers | Does the name resolve? Does the application respond correctly? | Use [[dig]] and inspect application or TLS errors |

Two common approaches are:

- **Bottom-up** — Start with the physical connection and work toward the application.
- **Top-down** — Start with the failing application and investigate its dependencies.

Choose the starting point based on the reported symptom and the available evidence.

### Example: A Website Does Not Open

1. Confirm that the interface is connected.
2. Verify the IP address, subnet prefix and default gateway.
3. Check the route and test IP reachability where ICMP is permitted.
4. Resolve the website's hostname and inspect the returned records.
5. Test the required TCP port, such as `443` for HTTPS over TCP.
6. If the connection succeeds, inspect TLS and HTTP errors.

> [!important] Understand the limits of each test
> A successful ping does not prove that DNS, a TCP port or an application works. A failed ping does not prove that the destination is unavailable, because ICMP may be filtered. A successful TCP connection does not prove that TLS negotiation or the application response will succeed.

See [[Network Connectivity Troubleshooting]] for the complete procedure and [[DNS Resolution Failure]] for a simulated troubleshooting case.

## 🔑 Key Points

- The OSI model has seven layers with distinct responsibilities.
- It is a reference model, not the protocol stack used by modern Internet hosts.
- Real protocols and devices may perform functions associated with multiple layers.
- Conventional Ethernet switching primarily operates at Layer 2.
- IP routing primarily operates at Layer 3.
- TCP and UDP provide Layer 4 transport functions.
- Troubleshooting requires interpreting each test within its scope.

## 🔗 Related Notes

- [[TCP-IP Model]]
- [[IP Addressing]]
- [[TCP and UDP]]
- [[DNS]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

## 📌 Notes

Use the OSI model as a diagnostic checklist. Record what each check establishes, what remains uncertain and which test should follow.

Reference: [ITU-T X.200 — OSI Basic Reference Model](https://www.itu.int/rec/T-REC-X.200/en/).