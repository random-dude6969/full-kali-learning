---
title: "siparmyknife Cheatsheet"
description: Quick reference for siparmyknife SIP vulnerability scanner
tool: siparmyknife
category: voip
difficulty: intermediate
---

# siparmyknife Cheatsheet

## Quick Start

```bash
# Basic scan
siparmyknife -t 192.168.1.100 -v

# User enumeration
siparmyknife -t 192.168.1.100 -d users.txt -v

# Single user test
siparmyknife -t 192.168.1.100 -u admin -v
```

## SIP Methods

| Method | Purpose |
|--------|---------|
| REGISTER | Register endpoint (enumerate users) |
| OPTIONS | Query capabilities |
| INVITE | Initiate call (verify users) |
| SUBSCRIBE | Subscribe to events |
| NOTIFY | Event notifications |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Target host | `-t 192.168.1.100` |
| `-p` | SIP port | `-p 5060` |
| `-d` | Dictionary file | `-d users.txt` |
| `-u` | Username | `-u admin` |
| `-v` | Verbose | `-v` |
| `-T` | Timeout | `-T 5` |
| `-f` | From domain | `-f evil.com` |
| `-m` | SIP method | `-m OPTIONS` |

## Examples

### Full Scan
```bash
siparmyknife -t 192.168.1.100 -d /usr/share/wordlists/sip_users.txt -v
```

### Custom Port
```bash
siparmyknife -t 192.168.1.100 -p 5080 -v
```

### Test Defaults
```bash
siparmyknife -t 192.168.1.100 -d defaults.txt -v
```

### Network Scan
```bash
for ip in $(seq 1 254); do
    siparmyknife -t 192.168.1.$ip -v 2>/dev/null
done
```

## Chaining

```bash
# Find SIP hosts, then enumerate
nmap -sU -p 5060 192.168.1.0/24 | grep "open" | awk '{print $2}' | while read ip; do
    siparmyknife -t $ip -d users.txt -v >> sip_results.txt
done

# Extract valid users
grep "Valid" sip_results.txt | awk '{print $2}' > valid_users.txt
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| No response | Check firewall on UDP 5060 |
| Timeout | Increase `-T` value |
| Permission denied | No root needed; check network |
| False positives | Verify with manual testing |

```bash
# Verify connectivity
nc -zvu 192.168.1.100 5060

# Capture for debugging
sudo tcpdump -i eth0 "udp port 5060 and host 192.168.1.100" -n
```
