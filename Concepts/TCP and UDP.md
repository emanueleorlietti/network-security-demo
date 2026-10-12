---
Concept: TCP and UDP
Description: "A comparison of TCP and UDP transport behavior, common use cases and troubleshooting considerations."
tags:
  - concept
  - networking
  - tcp
  - udp
  - transport
---

# 🧠 Concept

**Transmission Control Protocol (TCP)** and **User Datagram Protocol (UDP)** carry application traffic across IP networks.

Both use **port numbers** to identify transport endpoints. They operate at the Transport layer of the [[TCP-IP Model]], corresponding to Layer 4 of the [[OSI Model]].

Their main difference is the service they provide:

- **TCP** provides a reliable, ordered byte stream.
- **UDP** carries individual datagrams without built-in delivery or ordering guarantees.

## ⚖️ Comparison

| Property | TCP | UDP |
|---|---|---|
| Connection setup | Establishes a connection before normal data exchange | No transport handshake |
| Data representation | Byte stream | Separate datagrams |
| Delivery guarantee | Uses acknowledgments and retransmissions; reports connection failures | No delivery guarantee |
| Ordering | Delivers bytes in order | Datagrams may arrive out of order |
| Duplicate handling | Handles duplicate segments within the connection | Applications may receive duplicate datagrams |
| Flow control | Built in | Not provided by UDP |
| Congestion control | Built in | Must be addressed by the application or a protocol above UDP |
| Header size | At least 20 bytes | 8 bytes |
| Typical uses | HTTPS over TCP, SSH, SMTP | DNS queries, real-time media, QUIC |

> [!note] Reliability has limits
> TCP cannot guarantee that a remote application processes or saves the data. Applications need their own responses or acknowledgments to confirm those outcomes.

## 🤝 TCP Connection Establishment

TCP normally establishes a connection using a **three-way handshake**.

| Step | Direction | Purpose |
|---:|---|---|
| 1 | Client → Server: `SYN` | Requests a connection and announces the client's initial sequence number |
| 2 | Server → Client: `SYN, ACK` | Acknowledges the request and announces the server's initial sequence number |
| 3 | Client → Server: `ACK` | Acknowledges the server's sequence number |

Once established, the connection supports bidirectional data transfer.

TCP uses sequence numbers, acknowledgments and retransmissions to recover from lost data and deliver bytes in order.

### Byte Stream Behavior

TCP does not preserve application message boundaries.

For example, two application writes may be received together, or one write may be received in several reads. The application protocol must define how messages are separated, such as through lengths or delimiters.

### Connection Closure

TCP can close a connection gracefully using `FIN` and acknowledgment exchanges.

A `RST` resets the connection immediately. A reset can occur when a destination port has no listener or when an endpoint rejects an existing connection.

## 📬 UDP Datagram Delivery

UDP sends datagrams without first establishing a transport connection.

Each datagram preserves its own message boundary, but UDP does not:

- Retransmit lost datagrams.
- Reorder arriving datagrams.
- Confirm delivery.
- Provide flow or congestion control.

Applications can add these functions when needed.

> [!note] Protocols built on UDP
> QUIC uses UDP while adding secure connections, reliable streams and congestion control. A UDP-based application can therefore provide more guarantees than UDP itself.

### Datagram Size

Large datagrams may exceed the path's packet size limits.

Depending on the IP version and configuration, this can cause fragmentation or prevent delivery. Applications should account for path MTU and avoid assuming that very large datagrams will arrive successfully.

## 🔢 Port Numbers

TCP and UDP port numbers range from `0` to `65535`.

| Range | Category |
|---|---|
| `0–1023` | System ports |
| `1024–49151` | User ports |
| `49152–65535` | Dynamic or private ports |

Port `0` is reserved. Operating systems may use it in socket configuration to request automatic port assignment.

TCP and UDP have separate port spaces. A service can use the same numeric port with both protocols.

| Service | Common Transport | Typical Destination Port |
|---|---|---:|
| DNS | UDP and TCP | `53` |
| SSH | TCP | `22` |
| HTTP | TCP | `80` |
| HTTPS using HTTP/1.1 or HTTP/2 | TCP | `443` |
| HTTP/3 using QUIC | UDP | `443` |
| NTP | UDP | `123` |

These are conventional assignments. A port number alone does not prove which application is running.

## 🌐 Example: A DNS Query

Traditional DNS queries commonly start over UDP on port `53`.

If a response is marked as truncated, a client can retry the query over TCP. TCP also supports other DNS operations, including full zone transfers.

This means that permitting only UDP port `53` can produce partial DNS functionality.

See [[DNS]] for the complete resolution process and [[dig]] for query examples.

## 🛠️ Troubleshooting Considerations

| Observation | Possible Interpretation | Next Check |
|---|---|---|
| TCP connection succeeds | A transport connection reached an endpoint | Verify TLS and application behavior |
| TCP connection is refused | An endpoint actively rejected the connection | Check the listener, destination port and rejection policies |
| TCP connection times out | No connection completed within the timeout | Check routing, filtering, packet loss and destination availability |
| UDP query receives a valid reply | That specific request and response succeeded | Validate the response contents |
| UDP query receives no reply | The request, response or application behavior may have failed | Inspect both directions and test with a protocol-aware tool |

Use [[Test-NetConnection]] to test TCP connectivity to a specific port.

Use [[dig]] to test DNS behavior over UDP or TCP.

> [!important] UDP testing
> Sending a UDP datagram without receiving an error does not prove that the destination port is open. A protocol-aware request and a valid response provide stronger evidence.

## 🔑 Key Points

- TCP provides a reliable, ordered byte stream without preserving message boundaries.
- UDP preserves datagram boundaries without guaranteeing delivery or order.
- Neither protocol inherently encrypts application data.
- UDP is not automatically faster; performance depends on the application and network conditions.
- Port numbers identify transport endpoints, not the identity of an application.
- Successful transport connectivity does not prove that the application works.
- DNS requires consideration of both UDP and TCP.

## 🔗 Related Notes

- [[OSI Model]]
- [[TCP-IP Model]]
- [[IP Addressing]]
- [[DNS]]
- [[dig]]
- [[Test-NetConnection]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

## 📌 Notes

Identify the transport protocol before testing a service. A successful TCP test cannot establish that a UDP service using the same numeric port works.

References:

- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768.html)
- [RFC 8085 — UDP Usage Guidelines](https://www.rfc-editor.org/rfc/rfc8085.html)
- [RFC 7766 — DNS Transport over TCP](https://www.rfc-editor.org/rfc/rfc7766.html)
- [IANA — Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)