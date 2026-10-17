Technical cases organized around symptoms, context, investigation, verified root cause, resolution and validation.

Each case connects diagnostic evidence with the commands, concepts and procedures used to interpret it.

## 📋 All Cases

```dataview
TABLE WITHOUT ID
    file.link AS "Case",
    Description AS "Description",
    Status AS "Status",
    Date AS "Date"
FROM "Troubleshooting"
WHERE contains(tags, "troubleshooting")
SORT Date DESC, file.name ASC
```

## 🧭 Featured Case

Start with [[DNS Resolution Failure]], a simulated lab case involving an incorrect client DNS setting.

The investigation demonstrates how to:

- Separate name resolution from IP reachability.
- Compare configured and intended resolvers.
- Test HTTPS independently of DNS.
- Apply a targeted correction.
- Verify normal application access afterward.

## 💡 Reading a Case

- Begin with the reported problem and environment.
- Follow the evidence collected during the investigation.
- Distinguish observations from possible explanations.
- Check that the resolution addresses the identified cause.
- Review verification and prevention steps.

> [!note] Demo scope
> The current case is explicitly simulated. Its configuration and results illustrate a diagnostic method rather than document a production incident.

> [!note] Dynamic table
> The table requires Dataview and notes stored in `Troubleshooting` with the `troubleshooting` tag. It updates automatically as matching notes are added.

## 🔗 Navigation

- [[Dashboard]]
- [[Command Index]]
- [[Concept Index]]
- [[Guide Index]]
- [[Script Index]]
