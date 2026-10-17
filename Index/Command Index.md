Commands for inspecting network configuration, testing connectivity and investigating DNS behavior.

Each note includes syntax, common options, practical examples and interpretation limits.

## 🐧 Linux Commands

```dataview
TABLE WITHOUT ID
    file.link AS "Command",
    Description AS "Description"
FROM "Commands"
WHERE contains(tags, "command") AND (OS = "Linux" OR OS = "Linux / Windows")
SORT file.name ASC
```

## 🪟 Windows Commands

```dataview
TABLE WITHOUT ID
    file.link AS "Command",
    Description AS "Description"
FROM "Commands"
WHERE contains(tags, "command") AND (OS = "Windows" OR OS = "Linux / Windows")
SORT file.name ASC
```

## 🧭 Choose a Tool

| Task | Linux | Windows |
|---|---|---|
| Inspect IP configuration | [[ip]] | [[ipconfig]] |
| Inspect route selection | [[ip]] | [[Test-NetConnection]] |
| Test ICMP reachability | [[ping]] | [[Test-NetConnection]] |
| Investigate DNS responses | [[dig]] | [[Resolve-DnsName]] |
| Inspect HTTP/TLS behavior | [[curl]] | [[curl]] (use curl.exe) |
| Test a TCP port without testing the application | Use an appropriate TCP diagnostic tool | [[Test-NetConnection]] |

For a complete diagnostic sequence, follow [[Network Connectivity Troubleshooting]].

## Additional tools

- [[ss]] — local Linux sockets and listeners.
- [[curl]] — bounded HTTP/TLS diagnostics on Linux and Windows.
- [[Resolve-DnsName]] — explicit Windows DNS queries.

## 💡 Usage Notes

- Use the syntax documented for the relevant operating system.
- Record the destination, selected address and test result.
- Distinguish ICMP reachability, DNS resolution, TCP connectivity and application behavior.
- Review commands that change configuration before running them.

> [!note] Dynamic tables
> The tables require Dataview and the folder paths shown above. They update automatically as matching command notes are added.

## 🔗 Navigation

- [[Dashboard]]
- [[Concept Index]]
- [[Guide Index]]
- [[Script Index]]
- [[Troubleshooting Index]]
