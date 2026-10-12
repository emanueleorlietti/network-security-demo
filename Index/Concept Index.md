Networking concepts that explain how communication works and help interpret diagnostic results.

Each note connects the underlying theory with practical checks elsewhere in the vault.

## 📚 All Concepts

```dataview
TABLE WITHOUT ID
    file.link AS "Concept",
    Description AS "Description"
FROM "Concepts"
WHERE contains(tags, "concept")
SORT file.name ASC
```

## 🧭 Suggested Reading Order

| Order | Concept | Focus |
|---:|---|---|
| 1 | [[OSI Model]] | Organize communication functions into seven layers |
| 2 | [[TCP-IP Model]] | Understand how the Internet protocol suite fits together |
| 3 | [[IP Addressing]] | Interpret addresses, subnet prefixes and routing decisions |
| 4 | [[TCP and UDP]] | Distinguish transport behavior and testing requirements |
| 5 | [[DNS]] | Understand name resolution, records and resolver behavior |

## 🛠️ Apply the Concepts

Follow [[Network Connectivity Troubleshooting]] to turn these concepts into a diagnostic sequence.

Explore [[DNS Resolution Failure]] to see how evidence separates a name-resolution problem from an application or connectivity failure.

> [!note] Dynamic table
> The table requires Dataview and notes stored in `Concepts` with the `concept` tag. It updates automatically as matching notes are added.

## 🔗 Navigation

- [[Dashboard]]
- [[Command Index]]
- [[Guide Index]]
- [[Script Index]]
- [[Troubleshooting Index]]