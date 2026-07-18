---
title: "sipsak Cheatsheet"
description: "Quick reference for sipsak SIP testing tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: voip
categories:
  - cheatsheet
  - voip
  - sip
tags:
  - sipsak
  - sip
  - voip
  - quick-reference
---

# sipsak Cheatsheet

## Quick Start

```bash
# Basic OPTIONS probe
sipsak -s sip:user@192.168.1.100

# Verbose output
sipsak -v -s sip:user@192.168.1.100

# Register with credentials
sipsak -s sip:1000@192.168.1.100 -l 1000 -p password

# Quick scan with timeout
sipsak -q -T 2000 -s sip:user@192.168.1.100
```

## Common Flags

### Target

| Flag | Description | Example |
|------|-------------|---------|
| `-s` | SIP URI (required) | `-s sip:user@192.168.1.100` |
| `-H` | Override host | `-H 192.168.1.100` |
| `-O` | Outbound proxy | `-O 192.168.1.1:5060` |
| `-P` | SIP proxy | `-P proxy.local` |

### Authentication

| Flag | Description | Example |
|------|-------------|---------|
| `-l` | Username | `-l admin` |
| `-p` | Password | `-p secret` |
| `-a` | Auth realm | `-a pbx.local` |

### Method & Headers

| Flag | Description | Example |
|------|-------------|---------|
| `-m` | SIP method | `-m REGISTER` |
| `-I` | Max-Forwards | `-I 70` |
| `-C` | Content-Type | `-C "application/sdp"` |
| `-U` | User-Agent | `-U "Custom/1.0"` |

### Timing

| Flag | Description | Example |
|------|-------------|---------|
| `-T` | Timeout (ms) | `-T 5000` |
| `-M` | Max time (s) | `-M 60` |
| `-R` | Retransmissions | `-R 3` |
| `-B` | Bandwidth (pkt/s) | `-B 100` |
| `-Q` | Expires | `-Q 3600` |

### Output

| Flag | Description |
|------|-------------|
| `-v` | Verbose |
| `-VV` | Extra verbose |
| `-q` | Quiet |
| `-x` | Request count |

### Transport

| Flag | Description |
|------|-------------|
| `-6` | IPv6 |
| `-t tcp` | TCP transport |

## Examples

### Endpoint Discovery

```bash
# Probe single endpoint
sipsak -s sip:user@192.168.1.100

# Probe with custom port
sipsak -s sip:user@192.168.1.100:5061

# IPv6 target
sipsak -6 -s sip:user@::1
```

### Authentication Testing

```bash
# Test credentials
sipsak -s sip:1000@192.168.1.100 -l 1000 -p test123

# Wrong credentials (expect 401/403)
sipsak -v -s sip:1000@192.168.1.100 -l admin -p wrong

# Test with realm
sipsak -s sip:1000@192.168.1.100 -l 1000 -p ext -a company
```

### Method Testing

```bash
# OPTIONS (default)
sipsak -s sip:user@192.168.1.100

# REGISTER
sipsak -m REGISTER -s sip:1000@192.168.1.100 -l 1000 -p ext

# SUBSCRIBE
sipsak -m SUBSCRIBE -s sip:user@192.168.1.100

# MESSAGE
sipsak -m MESSAGE -s sip:user@192.168.1.100

# INVITE
sipsak -m INVITE -s sip:user@192.168.1.100
```

### Rate Control

```bash
# Slow scan (10 pkt/s)
sipsak -B 10 -s sip:user@192.168.1.100

# Fast scan (200 pkt/s)
sipsak -B 200 -s sip:user@192.168.1.100

# Limited requests
sipsak -x 5 -s sip:user@192.168.1.100
```

### Proxy Routing

```bash
# Via outbound proxy
sipsak -s sip:user@192.168.1.100 -O 10.0.0.1:5060

# Via SIP proxy
sipsak -s sip:user@pbx.local -P proxy.example.com
```

## Chaining

### With Other SIP Tools

```bash
# sipsak → sipvicious
sipsak -s sip:user@192.168.1.100 -v  # Initial probe
svmap 192.168.1.100                   # Detailed scan

# sipsak → nmap
sipsak -s sip:user@192.168.1.100     # SIP test
nmap -sU -p 5060 192.168.1.100      # Port verification
```

### With Network Tools

```bash
# Capture while testing
sipsak -s sip:user@192.168.1.100 &
tcpdump -i eth0 -w sip.pcap port 5060 &

# DNS + SIP
host sip.example.com && sipsak -s sip:user@sip.example.com
```

### Script Integration

```bash
# Quick scan multiple targets
for i in $(seq 1 254); do
    sipsak -q -s sip:user@192.168.1.$i 2>/dev/null && echo "192.168.1.$i active"
done

# Credential brute-force
while read pass; do
    sipsak -q -s sip:1000@192.168.1.100 -l 1000 -p "$pass" && echo "Found: $pass"
done < wordlist.txt
```

## Troubleshooting

### No Response

```bash
# Check connectivity
ping -c 3 192.168.1.100
nc -zvu 192.168.1.100 5060

# Increase timeout
sipsak -T 10000 -s sip:user@192.168.1.100

# Try TCP
sipsak -t tcp -s sip:user@192.168.1.100
```

### Authentication Errors

```bash
# Debug auth flow
sipsak -VV -s sip:1000@192.168.1.100 -l 1000 -p test

# Check realm from 401 response
sipsak -v -s sip:1000@192.168.1.100 -l 1000 -p test
```

### DNS Issues

```bash
# Use IP directly
sipsak -s sip:user@192.168.1.100

# Override DNS
sipsak -s sip:user@domain.com -H 192.168.1.100
```

### Return Codes

| Code | Meaning |
|------|---------|
| 0 | Success (200 OK) |
| 1 | No response |
| 2 | Non-200 response |
| 3 | Connection failed |
| 4 | DNS failed |
| 5 | Timeout |
| 6 | Bandwidth exceeded |

### Debug Commands

```bash
# Maximum verbosity
sipsak -VV -s sip:user@192.168.1.100

# Packet capture
sipsak -s sip:user@192.168.1.100 &
tcpdump -i eth0 -n port 5060 -A &
wait
```

---

**Last Updated**: 2026
**Tool Version**: sipsak 0.9.7
**Category**: VoIP Discovery
