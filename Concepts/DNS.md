---
Concept: DNS
Description: "How the Domain Name System resolves names, distributes records and supports network services."
tags:
  - concept
  - networking
  - dns
  - troubleshooting
---

# 🧠 Concept

The **Domain Name System (DNS)** is a distributed, hierarchical system that associates domain names with resource records.

Its best-known function is resolving names such as `www.example.com` to IPv4 or IPv6 addresses. DNS also publishes information used for email delivery, service discovery and other applications.

DNS operates at the Application layer of the [[TCP-IP Model]].

## 🧱 Main Components

| Component | Role |
|---|---|
| Stub resolver | Client-side component that sends queries to a recursive resolver |
| Recursive resolver | Retrieves answers for clients and caches results |
| Root nameserver | Directs queries toward the relevant top-level domain |
| TLD nameserver | Provides delegation information for domains under a top-level domain, such as `.com` |
| Authoritative nameserver | Serves records for zones it hosts |

A **zone** is a portion of the DNS namespace managed by an authority. A domain can delegate child domains to separate zones.

## 🔄 Name Resolution

For a lookup such as `www.example.com`:

1. The application requests name resolution through the client's resolver.
2. If no applicable local answer is available, the client queries a recursive resolver.
3. The recursive resolver checks its cache.
4. If necessary, it follows referrals from root, TLD and authoritative nameservers.
5. It retrieves the requested records, following aliases where needed.
6. It returns the result to the client and caches eligible information.

> [!note] Cached information
> A recursive resolver does not contact every level for every query. Cached answers and delegation information can shorten the process.

### Recursive and Iterative Queries

- **Recursive query** — Requests that the server obtain a final answer or report a failure.
- **Iterative query** — Allows the server to return its available answer or a referral to another server.

Clients commonly request recursion from their configured resolver. That resolver uses iterative queries when contacting authoritative infrastructure.

## 📋 Common Record Types

| Type | Purpose | Example |
|---|---|---|
| `A` | Maps a name to an IPv4 address | `www.example.com. A 192.0.2.10` |
| `AAAA` | Maps a name to an IPv6 address | `www.example.com. AAAA 2001:db8::10` |
| `CNAME` | Defines an alias to another name | `portal.example.com. CNAME www.example.com.` |
| `MX` | Identifies mail servers with preference values | `example.com. MX 10 mail.example.com.` |
| `NS` | Identifies authoritative nameservers | `example.com. NS ns1.example.com.` |
| `TXT` | Stores text used by applications and policies | `example.com. TXT "demo-record"` |
| `PTR` | Maps a reverse-lookup name to a hostname | `10.2.0.192.in-addr.arpa. PTR www.example.com.` |
| `SOA` | Contains zone metadata, including serial and timing values | Zone administration information |
| `SRV` | Identifies service targets, ports, priorities and weights | Service discovery information |

These examples are illustrative. The IP addresses use documentation ranges and do not represent working services.

A trailing dot indicates an **absolute DNS name**, also called a fully qualified domain name in this context.

## ⏱️ Caching and TTL

The **Time to Live (TTL)** determines how long a record may normally remain cached before it must be refreshed.

```text
www.example.com.  300  IN  A  192.0.2.10
```

Here, `300` is the TTL in seconds.

- Lower TTLs can allow changes to become visible sooner.
- Higher TTLs reduce repeated queries.
- Changing a record does not immediately remove answers already cached elsewhere.
- Negative answers can also be cached.

> [!important] DNS changes are not instant
> A resolver may continue returning an earlier answer until its cached entry expires. Lowering the authoritative TTL does not retroactively shorten an existing cached entry.

## 🚦 Responses and Failures

| Result | Meaning | What to Investigate |
|---|---|---|
| `NOERROR` with an answer | The query completed successfully and returned records | Confirm that the records are appropriate |
| `NOERROR` without the requested records | No records of that type may exist; inspect the response sections | Query name, record type, aliases or referrals |
| `NXDOMAIN` | The queried name does not exist in the responding DNS view | Spelling, search suffixes, zone data and resolver choice |
| `SERVFAIL` | The server could not complete the query | Upstream availability, delegation or DNSSEC validation |
| `REFUSED` | The server declined the query | Access policies and recursion permissions |
| Timeout | No usable response arrived before the deadline | Resolver reachability, filtering and server availability |

An empty Answer section alone does not establish that the name does not exist. The response code and other sections matter.

## 🚚 DNS Transport

Traditional DNS uses destination port `53` over both UDP and TCP.

| Transport | Common Use |
|---|---|
| UDP `53` | Ordinary queries when the response fits the supported size |
| TCP `53` | Queries, retries after truncated UDP responses and full zone transfers |
| DNS over TLS | Encrypted DNS, normally over TCP `853` |
| DNS over HTTPS | DNS carried over HTTPS, commonly on port `443` |

**EDNS** allows clients to advertise support for larger UDP responses. Large responses can still encounter fragmentation or path-size problems.

> [!important] Permit the required transports
> Allowing UDP port `53` while blocking TCP port `53` can cause intermittent or query-dependent failures.

See [[TCP and UDP]] for the underlying transport behavior.

## 🔐 DNS Security

Traditional DNS does not encrypt queries or responses.

**DNSSEC** provides authentication of signed DNS data and protection against undetected modification when validation succeeds. It does not encrypt the data.

Encrypted DNS transports protect communication between the client and the selected resolver. They do not, by themselves, prove that every returned record is authentic.

## 🛠️ Troubleshooting DNS

Separate DNS behavior from general network connectivity.

| Check | Purpose |
|---|---|
| Inspect client configuration | Identify the configured resolvers and search suffixes |
| Query a specific name and record type | Avoid ambiguity about the requested information |
| Query the intended resolver explicitly | Confirm which server produced the answer |
| Inspect status, flags and response sections | Distinguish missing records, refusals and server failures |
| Compare UDP and TCP behavior | Identify transport-specific problems |
| Compare relevant DNS views | Detect differences between internal and external answers |
| Check application behavior | Identify caching or resolver differences outside the test tool |

Use [[dig]] for explicit DNS queries and [[ipconfig]] to inspect Windows DNS configuration.

> [!note] Application resolution can differ
> Applications may use local hosts files, caches, search domains, VPN-specific DNS or their own encrypted resolver. A successful `dig` query does not prove that an application uses the same resolution path.

Internal and external resolvers can legitimately return different answers through **split DNS**. Compare results against the intended environment.

Follow [[Network Connectivity Troubleshooting]] for the general diagnostic sequence and [[DNS Resolution Failure]] for a simulated case.

## 🔑 Key Points

- DNS stores many record types, not just IP addresses.
- Recursive resolvers retrieve and cache answers for clients.
- Authoritative nameservers serve data for their zones.
- TTL affects caching and the visibility of changes.
- `NXDOMAIN`, `SERVFAIL`, `REFUSED` and timeouts describe different outcomes.
- Traditional DNS needs both UDP and TCP consideration.
- DNSSEC authenticates signed data; encrypted DNS protects the transport.
- Valid IP connectivity does not guarantee successful name resolution.

## 🔗 Related Notes

- [[TCP-IP Model]]
- [[IP Addressing]]
- [[TCP and UDP]]
- [[dig]]
- [[ipconfig]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

## 📌 Notes

Record the queried name, record type, resolver address, transport, response status and test time when documenting a DNS problem.

References:

- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034.html)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035.html)
- [RFC 2308 — Negative Caching of DNS Queries](https://www.rfc-editor.org/rfc/rfc2308.html)
- [RFC 7766 — DNS Transport over TCP](https://www.rfc-editor.org/rfc/rfc7766.html)
- [RFC 4033 — DNS Security Introduction and Requirements](https://www.rfc-editor.org/rfc/rfc4033.html)