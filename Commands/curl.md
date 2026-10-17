---
Command: "curl"
Description: "Separate HTTP, TLS, and connection failures with bounded requests."
OS: "Linux / Windows"
tags:
  - command
  - linux
  - windows
  - networking
  - http
  - tls
  - troubleshooting
---

## Purpose and execution context

Use a curl build supporting HTTPS; inspect `curl --version` (Windows: `curl.exe --version`). On Windows, invoke `curl.exe` explicitly to avoid PowerShell alias ambiguity. Requests need an authorized target and normally no elevation.

## Examples

```bash
curl --verbose --connect-timeout 5 --max-time 15 --output /dev/null https://example.com/
```

Windows equivalent:

```powershell
curl.exe --verbose --connect-timeout 5 --max-time 15 --output NUL https://example.com/
```

The connect timeout bounds connection establishment; max-time bounds the operation. Output is discarded while verbose diagnostics remain on stderr. See the guide for the lab-only `--resolve` test.

## Interpretation and limitations

Read connection, TLS and HTTP stages separately. Curl can return zero for an HTTP error status unless HTTP-failure handling is requested; check the response status as well as exit status. A response alone does not validate the intended user transaction.

## Impact, validation and cleanup

A request generates traffic and server logs. Verbose output may disclose headers and endpoints: redact before storing. Keep certificate validation enabled. Proxy, credentials, trust stores and redirects affect results; this example does not automatically follow redirects.

## Related notes

- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]
- [[TCP and UDP]]

## References and validation status

[curl official manual](https://curl.se/docs/manpage.html). Reviewed 2026-10-17; examples not executed against live services.
