---
title: "ike-scan Cheatsheet"
tool: "ike-scan"
---

# ike-scan Cheatsheet

## Quick Start

**Basic scan**:
```bash
sudo ike-scan 192.168.1.1
```

**Aggressive mode scan** (recommended for discovery):
```bash
sudo ike-scan -A 192.168.1.0/24
```

**PSK hash extraction**:
```bash
sudo ike-scan -P -A 192.168.1.1
```

## Scan Types

| Mode | Flag | Use Case |
|------|------|----------|
| Main Mode | `-M` | Default, stealthier |
| Aggressive Mode | `-A` | Faster, reveals more info |
| IKEv2 | `--ikev2` | Modern VPN implementations |
| PSK Crack | `-P` | Extract pre-shared key hash |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-A` | Aggressive mode | `ike-scan -A 192.168.1.0/24` |
| `-M` | Main mode (default) | `ike-scan -M 192.168.1.1` |
| `-P` | PSK crack attempt | `ike-scan -P -A 192.168.1.1` |
| `-d <port>` | Destination port | `ike-scan -d 4500 192.168.1.1` |
| `-s <port>` | Source port | `ike-scan -s 500 192.168.1.1` |
| `--retry=<n>` | Retries per host | `ike-scan --retry=5 192.168.1.1` |
| `--timeout=<ms>` | Response timeout | `ike-scan --timeout=1000 192.168.1.1` |
| `-f` | IKE fragmentation | `ike-scan -f -A 192.168.1.1` |
| `-v` | Verbose output | `ike-scan -v 192.168.1.1` |
| `-vv` | Very verbose | `ike-scan -vv 192.168.1.1` |
| `--showbackoff` | Timing analysis | `ike-scan --showbackoff 192.168.1.1` |

## Output Options

| Flag | Description |
|------|-------------|
| `-o <file>` | Save output to file |
| `--format=<fmt>` | Output format |

## Examples

**Scan for VPN endpoints**:
```bash
sudo ike-scan -A -v 192.168.1.0/24
```

**Test NAT-T port**:
```bash
sudo ike-scan -d 4500 192.168.1.1
```

**Custom identification payload**:
```bash
sudo ike-scan -A --id="testuser" 192.168.1.1
```

**Scan from target file**:
```bash
sudo ike-scan -l targets.txt -A
```

**Fast scan with low retry**:
```bash
sudo ike-scan --retry=1 --timeout=200 10.0.0.0/8
```

**IKEv2 specific scan**:
```bash
sudo ike-scan --ikev2 192.168.1.0/24
```

## Chaining

**With Nmap for IKE fingerprinting**:
```bash
sudo ike-scan -A 192.168.1.0/24 | grep -oP '\d+\.\d+\.\d+\.\d+' | \
    xargs -I {} nmap -sU -p 500 --script ike-version {}
```

**With hashcat for PSK cracking**:
```bash
sudo ike-scan -P -A 192.168.1.1 > psk_hash.txt
hashcat -m 5400 psk_hash.txt wordlist.txt
```

**With awk for host extraction**:
```bash
sudo ike-scan -A 192.168.1.0/24 | awk '/返回/{print $1}' > active_hosts.txt
```

## Troubleshooting

**No responses received**:
```bash
# Verify target is reachable
ping 192.168.1.1

# Try NAT-T port
sudo ike-scan -d 4500 192.168.1.1

# Increase timeout
sudo ike-scan --timeout=2000 192.168.1.1

# Check firewall rules
sudo iptables -L -n | grep -E "500|4500"
```

**Permission errors**:
```bash
# Run with sudo
sudo ike-scan 192.168.1.1

# Or set capabilities
sudo setcap cap_net_raw+ep /usr/bin/ike-scan
```

**Inconsistent results**:
```bash
# Analyze timing with backoff
sudo ike-scan --showbackoff 192.168.1.1

# Reduce rate
sudo ike-scan --timeout=1000 --retry=3 192.168.1.1
```
