---
Command: "ss"
Description: "Inspect local socket state without probing a remote service."
OS: "Linux"
tags:
  - command
  - linux
  - networking
  - tcp
  - udp
  - troubleshooting
---

## Purpose and execution context

Linux iproute2, in the relevant network namespace. Ordinary inspection works without elevation; process ownership information may require extra privileges.

## Examples

```bash
ss -tuln
ss -tn state established
ss -ltnp
```

`-t` selects TCP, `-u` UDP, `-l` listening sockets, `-n` numeric output and `-p` process information.

## Interpretation and limitations

Check local address/port and protocol. A listener bound to loopback is different from one bound to all interfaces. A local socket entry does not prove remote reachability or successful application processing. UDP has no TCP-style listening handshake.

## Impact and validation

Local inspection without connection probes or configuration changes; no rollback required. Redact socket endpoints and process details before sharing. Validate the actual client path separately.

## Related notes

- [[TCP and UDP]]
- [[ip]]
- [[network-diagnostics]]

## References and validation status

[iproute2 ss manual](https://man7.org/linux/man-pages/man8/ss.8.html). Reviewed 2026-10-17; examples not executed on a Linux host.
