---
title: "udptunnel CHEATSHEET"
description: "Quick reference for udptunnel UDP-over-TCP tunneling commands"
---

# udptunnel CHEATSHEET

## Basic Usage

```bash
# Start server
udptunnel -s -p 5000

# Start client
udptunnel -c SERVER:5000 -l 5353 -d TARGET:53

# Test tunnel
dig @127.0.0.1 -p 5353 example.com
```

## Common Configurations

```bash
# DNS tunnel
udptunnel -s -p 5000                                  # Server
udptunnel -c SERVER:5000 -l 5353 -d DNS:53            # Client

# SIP tunnel
udptunnel -s -p 5000                                  # Server
udptunnel -c SERVER:5000 -l 5060 -d SIP:5060          # Client

# NTP tunnel
udptunnel -s -p 5000                                  # Server
udptunnel -c SERVER:5000 -l 123 -d NTP:123            # Client

# With bandwidth limit
udptunnel -c SERVER:5000 -l 5353 -d DNS:53 -b 10000
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-s` | Server mode |
| `-c SERVER:PORT` | Client connect to server |
| `-p PORT` | Listen port (server) |
| `-l PORT` | Local UDP listen port (client) |
| `-d HOST:PORT` | Destination address (client) |
| `-b BYTES` | Bandwidth limit |

## Practical Workflow

```bash
# 1. Start server on public IP
udptunnel -s -p 5000

# 2. Start client behind firewall
udptunnel -c PUBLIC_IP:5000 -l 5353 -d INTERNAL_DNS:53

# 3. Use tunneled DNS
dig @127.0.0.1 -p 5353 example.com
```

## Troubleshooting

```bash
# Check if running
ps aux | grep udptunnel

# Check port
ss -tlnp | grep 5000

# Kill tunnel
pkill udptunnel

# Test connectivity
nc -zv SERVER 5000
```
