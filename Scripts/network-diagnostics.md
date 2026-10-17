---
Script: "network-diagnostics"
Description: "Collects Linux network state and performs bounded ICMP and name-resolution checks without changing configuration."
Language: "Bash"
tags:
  - script
  - linux
  - networking
  - diagnostics
  - troubleshooting
---

# 🧩 Description

This Bash script collects a diagnostic snapshot to support [[Network Connectivity Troubleshooting]].

It displays:

- System information.
- Interface state and IP addresses.
- IPv4 and IPv6 routes.
- The selected IPv4 route toward a numeric target.
- Neighbor entries.
- Resolver configuration.
- ICMP test results.
- System name-resolution results.
- Explicit DNS query results when `dig` is available.
- Listening TCP and UDP sockets.

Checks continue after individual failures. Optional tools are detected before use.

> [!note] Scope
> The script gathers evidence rather than producing an automatic diagnosis. It does not change network configuration, flush caches or release DHCP leases. Queries can populate caches and generate logs.

# ⚙️ Script Content

```bash
#!/usr/bin/env bash

set -u
set -o pipefail

TARGET="${1:-1.1.1.1}"
DNS_NAME="${2:-example.com.}"
DNS_SERVER="${3:-}"

usage() {
    printf 'Usage: %s [IPv4-target] [DNS-name] [DNS-server-IP]\n' "$0"
}

section() {
    printf '\n========== %s ==========\n' "$1"
}

have() {
    command -v "$1" >/dev/null 2>&1
}

valid_ipv4() {
    local address="$1"
    local octet
    local -a octets

    [[ "$address" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1

    IFS='.' read -r -a octets <<< "$address"

    for octet in "${octets[@]}"; do
        (( 10#$octet <= 255 )) || return 1

        # Avoid ambiguous addresses with leading zeros.
        [[ "$octet" == "0" || "$octet" != 0* ]] || return 1
    done
}

run_check() {
    local label="$1"
    local result

    shift
    printf '\n-- %s --\n' "$label"

    # GNU timeout bounds each individual command.
    timeout --kill-after=2s 15s "$@"
    result=$?

    case "$result" in
        0)
            ;;
        124|137)
            printf '[TIMEOUT] The command exceeded its time limit.\n'
            ;;
        *)
            printf '[NOTICE] Command exit status: %s\n' "$result"
            ;;
    esac

    # Continue collecting evidence after failed checks.
    return 0
}

if (( $# > 3 )); then
    usage
    exit 2
fi

if ! valid_ipv4 "$TARGET"; then
    printf 'Error: the connectivity target must be a dotted-decimal IPv4 address.\n' >&2
    exit 2
fi

if [[ -z "$DNS_NAME" || "$DNS_NAME" == -* || "$DNS_NAME" == +* ||
      "$DNS_NAME" == @* || "$DNS_NAME" =~ [[:space:]] ]]; then
    printf 'Error: provide a DNS name without whitespace or option prefixes.\n' >&2
    exit 2
fi

if [[ -n "$DNS_SERVER" ]]; then
    # Accept an IPv4 address or an IPv6-like numeric literal.
    # dig performs the final validation of IPv6 syntax.
    if ! valid_ipv4 "$DNS_SERVER" &&
       ! [[ "$DNS_SERVER" == *:* &&
            "$DNS_SERVER" =~ ^[0-9A-Fa-f:]+$ ]]; then
        printf 'Error: provide a numeric IPv4 or IPv6 DNS server address.\n' >&2
        exit 2
    fi
fi

for required_tool in ip timeout; do
    if ! have "$required_tool"; then
        printf 'Error: required tool unavailable: %s\n' "$required_tool" >&2
        exit 1
    fi
done

section "System Information"
date -u '+UTC timestamp: %Y-%m-%dT%H:%M:%SZ'
uname -sr

if have hostname; then
    hostname
fi

section "Interface State"
run_check "Link overview" ip -brief link show
run_check "Address overview" ip -brief address show

section "Routing"
run_check "IPv4 routes" ip -4 route show
run_check "IPv6 routes" ip -6 route show
run_check "Selected route to ${TARGET}" ip -4 route get "$TARGET"

section "Neighbor Entries"
run_check "Neighbor table" ip neigh show

section "Resolver Configuration"

if [[ -r /etc/resolv.conf ]]; then
    run_check "/etc/resolv.conf" cat /etc/resolv.conf
else
    printf '[SKIP] /etc/resolv.conf is not readable.\n'
fi

if have resolvectl; then
    run_check "Per-link resolver information" resolvectl status
else
    printf '[SKIP] resolvectl is unavailable.\n'
fi

section "ICMP Reachability"

if have ping; then
    # Select a conventional default gateway from the main table.
    # This is not a complete policy-routing analysis.
    gateway=""

    while read -r -a route_fields; do
        for ((i = 0; i + 1 < ${#route_fields[@]}; i++)); do
            if [[ "${route_fields[i]}" == "via" ]]; then
                gateway="${route_fields[i + 1]}"
                break
            fi
        done

        [[ -n "$gateway" ]] && break
    done < <(timeout --kill-after=2s 5s ip -4 route show default)

    if [[ -n "$gateway" ]]; then
        run_check "Default gateway ${gateway}" \
            ping -4 -n -c 3 -W 2 -w 8 "$gateway"
    else
        printf '[SKIP] No conventional IPv4 default gateway found in the main table.\n'
    fi

    run_check "IPv4 target ${TARGET}" \
        ping -4 -n -c 3 -W 2 -w 8 "$TARGET"
else
    printf '[SKIP] ping is unavailable.\n'
fi

section "System Name Resolution"

if have getent; then
    run_check "System lookup for ${DNS_NAME}" \
        getent ahosts "$DNS_NAME"
else
    printf '[SKIP] getent is unavailable.\n'
fi

section "DNS Queries"

if have dig; then
    dig_server=()

    if [[ -n "$DNS_SERVER" ]]; then
        dig_server=("@${DNS_SERVER}")
        printf 'Explicit resolver: %s\n' "$DNS_SERVER"
    else
        printf 'Using the default resolver configuration selected by dig.\n'
    fi

    run_check "A query, UDP with normal TCP fallback" \
        dig "${dig_server[@]}" "$DNS_NAME" A +time=2 +tries=1

    run_check "AAAA query, UDP with normal TCP fallback" \
        dig "${dig_server[@]}" "$DNS_NAME" AAAA +time=2 +tries=1

    run_check "A query over TCP" \
        dig "${dig_server[@]}" "$DNS_NAME" A +tcp +time=2 +tries=1
else
    printf '[SKIP] dig is unavailable; explicit DNS checks were not performed.\n'
fi

section "Listening Sockets"

if have ss; then
    run_check "Listening TCP and UDP sockets" ss -tuln
else
    printf '[SKIP] ss is unavailable.\n'
fi

section "Collection Complete"
printf 'Review errors, skipped checks, DNS status codes and selected routes.\n'
printf 'Completion does not mean that connectivity or DNS is healthy.\n'
```

# ▶️ Usage

Save the script content as `network-diagnostics.sh`.

Check Bash syntax and make it executable:

```bash
bash -n network-diagnostics.sh
chmod +x network-diagnostics.sh
```

## Use Default Targets

```bash
./network-diagnostics.sh
```

Defaults:

- ICMP target: `1.1.1.1`.
- Name-resolution target: `example.com.`.
- DNS query server: the configuration used by `dig`.

## Use the Demo Lab

```bash
./network-diagnostics.sh 192.168.10.50 portal.demo.test. 192.168.10.53
```

This requires the fictional lab described in [[Network Connectivity Troubleshooting]].

The explicit resolver applies only to `dig` queries. The `getent` check continues to use the system's normal resolution configuration.

That comparison can help expose a difference between the configured resolution path and the intended DNS server.

## Capture a Report

```bash
./network-diagnostics.sh 192.168.10.50 portal.demo.test. 192.168.10.53 > network-report.txt 2>&1
```

Run without `sudo` unless elevated access is specifically needed.

# 🔒 Requirements

Required:

- Linux.
- Bash.
- `iproute2` for `ip`.
- GNU Coreutils `timeout`.

Optional:

- iputils `ping`.
- `getent` for system name resolution.
- BIND `dig` for explicit DNS queries.
- `resolvectl` for systemd-resolved information.
- `ss`, normally supplied by `iproute2`.

Package names vary by distribution.

# 💡 Notes

- The ICMP target must be a numeric IPv4 address.
- The gateway check uses the first conventional default route found in the main IPv4 table.
- VPNs, multiple routes and policy routing require additional analysis.
- An ICMP failure does not prove that an application is unavailable.
- `getent` can use hosts files and other configured name services; `dig` queries DNS.
- A zero exit status from `dig` does not establish that a requested record exists. Inspect the DNS response status and sections.
- Listening sockets describe the local host, not remote service availability.
- The script exits successfully after collection even if individual checks fail. Its exit status is not a health indicator.
- Commands inspect the current network namespace.
- Reports can contain hostnames, addresses, MAC addresses and internal DNS details. Review and redact them before public sharing.

Related notes:

- [[ip]]
- [[ping]]
- [[dig]]
- [[IP Addressing]]
- [[DNS]]
- [[Network Connectivity Troubleshooting]]
- [[DNS Resolution Failure]]

## Exit status and validation boundary

The script exits with status 2 for rejected arguments and status 1 when a required tool is missing. Individual checks report errors/timeouts and continue; an overall zero exit status means collection reached the end, not that every check passed. Optional tools may be skipped. Stop with Ctrl+C. The script writes to standard output/error; a report file is created only by the caller's redirection and may overwrite an existing file. Review reports for hostnames, addresses, resolver/search domains and listening ports before sharing.

The IPv4 target is validated; an optional IPv6 resolver literal is checked only superficially and then interpreted by dig. This is not a full IPv6 resolver-address validator.
