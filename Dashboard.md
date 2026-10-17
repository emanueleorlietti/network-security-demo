# Network & Security Demo

A public, vendor-neutral Obsidian demonstration of structured technical documentation. Current content focuses on networking and Linux/Windows diagnostics. Lab outcomes are simulated unless a note explicitly records an executed test.

## Browse

- [[Command Index]] — 8 command notes, including Linux/Windows HTTP diagnostics.
- [[Concept Index]] — 5 networking fundamentals with a suggested reading order.
- [[Guide Index]] — an end-to-end connectivity investigation.
- [[Script Index]] — a bounded Linux diagnostic collector.
- [[Troubleshooting Index]] — an explicitly simulated DNS case.
- [[README]] — setup, templates, Git synchronization and keyboard shortcuts.

## Start with a question

| Question | Start here | Supporting tools |
| --- | --- | --- |
| How do network layers fit together? | [[OSI Model]], [[TCP-IP Model]] | [[IP Addressing]], [[TCP and UDP]] |
| Why does a connection fail? | [[Network Connectivity Troubleshooting]] | [[ip]], [[ipconfig]], [[ping]], [[Test-NetConnection]] |
| Is the problem DNS? | [[DNS]], [[DNS Resolution Failure]] | [[dig]], [[Resolve-DnsName]] |
| Is there a local listener or an HTTP/TLS error? | [[ss]], [[curl]] | [[TCP and UDP]] |
| How can I collect supporting evidence? | [[network-diagnostics]] | [[Network Connectivity Troubleshooting]] |

## Create a note

Press Ctrl/Cmd+R for the configured QuickAdd actions: Linux command, Windows command, concept, guide, script or troubleshooting case. For a cross-platform command, choose a command action then document both contexts and set OS to Linux / Windows. Keep the command tag and relevant platform tags.

## Recently modified

```dataview
TABLE Description AS "Purpose", file.mtime AS "Modified"
FROM "Commands" OR "Concepts" OR "Guides" OR "Scripts" OR "Troubleshooting"
SORT file.mtime DESC
LIMIT 10
```

The table requires Dataview. Static links above work without it. File modification time is not a test date. Record observations, hypotheses and verified outcomes separately; review every excerpt before publishing it.
