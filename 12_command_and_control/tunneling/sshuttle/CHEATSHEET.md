---
title: "sshuttle CHEATSHEET"
description: "Quick reference for sshuttle transparent VPN over SSH commands"
---

# sshuttle CHEATSHEET

## Basic Usage

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8          # Route subnet
sshuttle -r user@attacker_ip 0/0                  # Route all traffic
sshuttle -r user@attacker_ip --auto-nets          # Auto-detect subnets
```

## Subnet Routing

```bash
# Multiple subnets
sshuttle -r user@attacker_ip 10.0.0.0/8 192.168.0.0/16

# Exclude subnet
sshuttle -r user@attacker_ip 10.0.0.0/8 -x 10.0.1.0/24

# Subnet file
sshuttle -r user@attacker_ip -s subnets.txt
```

## DNS Tunneling

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns       # Forward DNS
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns --to-ns 8.8.8.8  # Custom DNS
```

## SSH Options

```bash
# Custom SSH command
sshuttle -r user@attacker_ip 10.0.0.0/8 -e "ssh -i key -p 2222"

# Via proxy
sshuttle -r user@attacker_ip 10.0.0.0/8 -e "ssh -o ProxyCommand='connect-proxy PROXY:8080 %h %p'"
```

## Connection Methods

```bash
sshuttle --method nat -r user@attacker_ip 10.0.0.0/8      # NAT (default)
sshuttle --method tproxy -r user@attacker_ip 10.0.0.0/8   # TProxy
sshuttle --method pf -r user@attacker_ip 10.0.0.0/8       # PF (BSD)
```

## Advanced Features

```bash
# Daemon mode
sshuttle -r user@attacker_ip 10.0.0.0/8 -D

# Auto hostname scanning
sshuttle -r user@attacker_ip 10.0.0.0/8 --auto-hosts

# No latency control
sshuttle -r user@attacker_ip 10.0.0.0/8 --no-latency-control

# Custom PID file
sshuttle -r user@attacker_ip 10.0.0.0/8 -D --pidfile /tmp/sshuttle.pid
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-r USER@HOST` | Remote SSH server |
| `-x SUBNET` | Exclude subnet |
| `-s FILE` | Subnet file |
| `-e CMD` | Custom SSH command |
| `--dns` | Forward DNS |
| `--to-ns IP` | DNS server for queries |
| `--auto-nets` | Auto-detect subnets |
| `--auto-hosts` | Auto-scan hostnames |
| `--method METHOD` | Connection method |
| `-D` | Daemon mode |
| `--no-latency-control` | Disable latency control |

## Practical Workflow

```bash
# 1. SSH to pivot host
sshuttle -r user@pivot_host 10.10.10.0/24

# 2. Access internal services
ssh user@10.10.10.5
curl http://10.10.10.10
nmap -sT 10.10.10.0/24

# 3. Chain through multiple pivots
sshuttle -r user@10.10.10.5 172.16.0.0/16

# 4. Run as daemon
sshuttle -r user@pivot_host 10.10.10.0/24 -D --pidfile /tmp/sshuttle.pid
```

## Troubleshooting

```bash
# Check if running
ps aux | grep sshuttle

# Kill sshuttle
pkill sshuttle

# Check routes
ip route show

# Test connectivity
ping 10.10.10.1
```
