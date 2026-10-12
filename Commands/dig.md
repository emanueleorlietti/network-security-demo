---
Command: dig
Description: "Queries DNS servers and displays records, response status and diagnostic information."
OS: Linux
tags:
  - command
  - linux
  - networking
  - dns
  - troubleshooting
---

# 🛠️ Command

```bash
dig [@SERVER] NAME [TYPE] [OPTIONS]
```

The **dig** command, provided by BIND utilities, sends DNS queries and displays the responses.

It helps inspect:

- Resource records.
- The responding server.
- Response codes and flags.
- Query duration.
- Differences between resolvers or transports.

By default, `dig` requests an `A` record and normally uses a nameserver from `/etc/resolv.conf`.

# 🔧 Common Options

| Option or Argument | Purpose |
|---|---|
| `@SERVER` | Sends the query to a specific DNS server |
| `A`, `AAAA`, `MX`, etc. | Selects the requested record type |
| `+short` | Displays concise answer information |
| `+noall +answer` | Displays only the Answer section |
| `+tcp` | Uses TCP instead of the default UDP query |
| `+time=N` | Sets the query timeout in seconds |
| `+tries=N` | Sets the number of attempts for UDP queries |
| `-x ADDRESS` | Performs a reverse lookup |
| `+trace` | Follows DNS delegations from the root |
| `+dnssec` | Requests DNSSEC records by setting the DO bit |

# 📚 Examples

## Query an IPv4 Address Record

```bash
dig example.com. A
```

The trailing dot makes the name absolute.

Keep the full output when investigating a failure: status, flags and response sections provide important context.

## Query an IPv6 Address Record

```bash
dig example.com. AAAA
```

An `A` answer does not establish that an `AAAA` record exists, or that IPv6 connectivity works.

## Query a Specific Resolver

```bash
dig @192.168.10.53 example.com. A
```

Replace `192.168.10.53` with the intended resolver address.

Using a numeric server address avoids needing to resolve the server's hostname before performing the test.

## Display Concise Answers

```bash
dig example.com. A +short
```

This is useful for quick inspection, but hides diagnostic details.

No output from `+short` does not distinguish between a missing record, a nonexistent name or a query failure. Repeat the query without `+short`.

## Display Only the Answer Section

```bash
dig example.com. A +noall +answer
```

Answer rows normally contain:

```text
NAME  TTL  CLASS  TYPE  VALUE
```

For example, a simulated lab record could appear as:

```text
portal.demo.test.  300  IN  A  192.168.10.50
```

This is illustrative output from a fictional lab zone, not a public DNS record.

## Query Other Record Types

```bash
dig example.com. MX
dig example.com. NS
dig example.com. TXT
dig example.com. SOA
```

Specify the record type relevant to the investigation rather than assuming an `A` lookup answers every DNS question.

## Compare UDP and TCP

```bash
dig @192.168.10.53 example.com. A +time=2 +tries=1
dig @192.168.10.53 example.com. A +tcp +time=2
```

If the results differ, investigate transport filtering, server behavior and path limitations.

> [!note] Automatic TCP retry
> A normal UDP query may automatically retry over TCP when the response is marked as truncated. Do not assume that a successful default query proves UDP alone returned the complete answer.

## Perform a Reverse Lookup

```bash
dig @192.168.10.53 -x 192.168.10.50
```

For IPv4, `dig` constructs the corresponding name under `in-addr.arpa`. For IPv6, it uses `ip6.arpa`.

A missing PTR record does not prevent ordinary forward resolution or IP connectivity.

## Inspect Delegation

```bash
dig example.com. A +trace
```

This follows referrals from the root and contacts the relevant servers directly.

It requires access to external DNS servers and can fail on networks that only permit DNS through approved recursive resolvers.

It does not reproduce the configured resolver's cache or internal DNS view.

## Request DNSSEC Information

```bash
dig example.com. A +dnssec
```

This requests DNSSEC-related records.

> [!important] Requesting is not validating
> `+dnssec` does not make `dig` independently validate the answer. An `ad` flag indicates that the responding resolver reports authenticated data; interpreting that claim depends on trusting the resolver and the connection to it.

# 🩺 Interpreting the Response

| Field | What to Look For |
|---|---|
| `status` | `NOERROR`, `NXDOMAIN`, `SERVFAIL`, `REFUSED` or another response code |
| `flags` | Response properties such as authoritative answer or recursion availability |
| `QUESTION SECTION` | The actual name and record type queried |
| `ANSWER SECTION` | Returned records, including aliases |
| `AUTHORITY SECTION` | Authority information, referrals or an SOA for negative responses |
| `ADDITIONAL SECTION` | Supporting information, including EDNS details |
| `SERVER` | The server that answered |
| `Query time` | Time taken for the query; not application response time |

Common flags include:

| Flag | Meaning |
|---|---|
| `qr` | The message is a response |
| `aa` | The response is authoritative |
| `rd` | Recursion was requested |
| `ra` | Recursion is available |
| `tc` | The response is truncated |
| `ad` | The resolver reports authenticated data |

`NOERROR` does not necessarily mean the requested record exists. Inspect the Answer section and the rest of the response.

# 💡 Notes

- Package names vary by distribution; common examples include `dnsutils` and `bind-utils`.
- `/etc/resolv.conf` may point to a local resolver stub rather than directly to an upstream server.
- `dig` queries DNS; it does not use the full system name-resolution path, such as hosts-file lookup.
- Applications may use different caches, resolver settings or encrypted DNS.
- Public resolvers generally cannot answer private internal zones.
- Compare resolvers only when their DNS views are expected to match.
- A DNS answer does not prove that the returned address or application is reachable.
- For standard queries, an exit status of zero can include an `NXDOMAIN` response. Inspect the DNS status, not just the shell exit code.
- Avoid querying sensitive internal names through external resolvers.

# 🔗 Related Notes

- [[DNS]]
- [[TCP and UDP]]
- [[IP Addressing]]
- [[ping]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

Reference: Run `man dig` for the documentation installed on your system.