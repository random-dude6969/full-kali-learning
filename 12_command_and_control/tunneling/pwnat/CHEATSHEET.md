---
title: "pwnat CHEATSHEET"
description: "Quick reference for pwnat NAT traversal commands"
---

# pwnat CHEATSHEET

## Basic Usage

```bash
# Start airlock server
pwnat -s -p 35000

# Connect client
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22

# SSH through tunnel
ssh -p 8080 user@127.0.0.1
```

## Common Configurations

```bash
# SSH tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8000 -r TARGET:22

# HTTP tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:80

# RDP tunnel
pwnat -c AIRLOCK:35000 -l 127.0.0.1:3389 -r TARGET:3389

# Verbose mode
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22 -v

# Daemon mode
pwnat -c AIRLOCK:35000 -l 127.0.0.1:8080 -r TARGET:22 -d
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-s` | Start server (airlock) |
| `-p PORT` | Listen/connect port |
| `-c SERVER` | Connect to airlock |
| `-l ADDR:PORT` | Local listen address |
| `-r ADDR:PORT` | Remote target address |
| `-i IFACE` | Network interface |
| `-v` | Verbose mode |
| `-d` | Daemon mode |

## Practical Workflow

```bash
# 1. Start airlock on public VPS
pwnat -s -p 35000

# 2. Client behind NAT
pwnat -c VPS_IP:35000 -l 127.0.0.1:8000 -r TARGET:22

# 3. SSH through tunnel
ssh -p 8000 user@127.0.0.1

# 4. Create SOCKS proxy
ssh -D 1080 -p 8000 user@127.0.0.1

# 5. Scan through proxy
proxychains4 nmap -sT 10.10.10.0/24
```

## Troubleshooting

```bash
# Test UDP connectivity
nc -u AIRLOCK_IP 35000

# Check local port
ss -tlnp | grep 8080

# Kill pwnat
pkill pwnat

# Check server
pwnat -s -p 35000 -v
```

## Limitations

- UDP only (not TCP directly)
- Requires public airlock server
- Does not work with symmetric NAT
- No encryption (use SSH/TLS)
- Initial connection may be slow
