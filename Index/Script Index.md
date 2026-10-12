Diagnostic and automation scripts with documented requirements, usage examples and limitations.

Each note includes the script content and explains how to interpret its results.

## 📋 All Scripts

```dataview
TABLE WITHOUT ID
    file.link AS "Script",
    Description AS "Description",
    Language AS "Language"
FROM "Scripts"
WHERE contains(tags, "script")
SORT file.name ASC
```

## 🧭 Where to Start

Use [[network-diagnostics]] to collect a Linux network snapshot, including:

- Interfaces and IP addresses.
- Routes and neighbor entries.
- Resolver configuration.
- ICMP and name-resolution checks.
- Listening TCP and UDP sockets.

Interpret the collected evidence using [[Network Connectivity Troubleshooting]].

## 💡 Using a Script

- Review the content and requirements before execution.
- Adapt targets to the intended environment.
- Check reported errors and skipped tests.
- Distinguish successful collection from healthy connectivity.
- Review captured reports for sensitive information before sharing.

> [!note] Dynamic table
> The table requires Dataview and notes stored in `Scripts` with the `script` tag. It updates automatically as matching notes are added.

## 🔗 Navigation

- [[Dashboard]]
- [[Command Index]]
- [[Concept Index]]
- [[Guide Index]]
- [[Troubleshooting Index]]