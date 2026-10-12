Reusable technical procedures with prerequisites, diagnostic steps, verification and rollback considerations.

Guides connect individual commands and concepts into a structured workflow.

## 📋 All Guides

```dataview
TABLE WITHOUT ID
    file.link AS "Guide",
    Description AS "Description",
    Difficulty AS "Difficulty"
FROM "Guides"
WHERE contains(tags, "guide")
SORT file.name ASC
```

## 🧭 Where to Start

Begin with [[Network Connectivity Troubleshooting]] to investigate a failed connection.

The procedure separates:

- Local interface and IP configuration.
- Routing and next-hop connectivity.
- DNS resolution.
- Transport connectivity.
- Application behavior.

Use [[network-diagnostics]] to collect supporting Linux diagnostic information, and explore [[DNS Resolution Failure]] for a simulated application of the procedure.

## 💡 Using a Guide

- Confirm the prerequisites and adapt lab values to your environment.
- Collect evidence before changing configuration.
- Interpret each test within its scope.
- Record original settings before applying a correction.
- Verify the original user workflow after the change.

> [!note] Dynamic table
> The table requires Dataview and notes stored in `Guides` with the `guide` tag. It updates automatically as matching notes are added.

## 🔗 Navigation

- [[Dashboard]]
- [[Command Index]]
- [[Concept Index]]
- [[Script Index]]
- [[Troubleshooting Index]]