---
title: "proxytunnel CHEATSHEET"
description: "Quick reference for proxytunnel HTTP CONNECT tunneling commands"
---

# proxytunnel CHEATSHEET

## Basic Usage

```bash
# Create tunnel
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222

# With authentication
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -P user:password

# SSL to proxy
proxytunnel -p PROXY:443 -d TARGET:22 -a 127.0.0.1:2222 -S

# Verbose mode
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -v
```

## SSH Tunneling

```bash
# Create tunnel
proxytunnel -p PROXY:8080 -d SSH_HOST:22 -a 127.0.0.1:2222

# Connect
ssh -p 2222 user@127.0.0.1

# SSH ProxyCommand
ssh -o ProxyCommand='proxytunnel -p PROXY:8080 -d %h:%p' user@target
```

## VNC/RDP

```bash
# VNC
proxytunnel -p PROXY:8080 -d VNC_HOST:5900 -a 127.0.0.1:5900
vncviewer 127.0.0.1:5900

# RDP
proxytunnel -p PROXY:8080 -d RDP_HOST:3389 -a 127.0.0.1:3389
rdesktop 127.0.0.1:3389
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-p PROXY:PORT` | Proxy address |
| `-d HOST:PORT` | Destination address |
| `-a ADDR:PORT` | Local listen address |
| `-P USER:PASS` | Proxy authentication |
| `-S` | SSL to proxy |
| `-N` | NTLM authentication |
| `-u USER` | NTLM username |
| `-s PASS` | NTLM password |
| `-d DOMAIN` | NTLM domain |
| `-H "K:V"` | Custom header |
| `-v` | Verbose mode |

## Practical Examples

```bash
# SSH to internal server
proxytunnel -p corp-proxy:8080 -d 10.0.0.5:22 -a 127.0.0.1:2222 -P user:pass
ssh -p 2222 user@127.0.0.1

# Web access
proxytunnel -p corp-proxy:8080 -d webserver:80 -a 127.0.0.1:8080 -P user:pass
curl http://127.0.0.1:8080

# Multi-hop
proxytunnel -p PROXY:8080 -d PIVOT:22 -a 127.0.0.1:2222 -P user:pass
ssh -D 1080 -p 2222 user@127.0.0.1
proxychains4 nmap -sT 10.10.10.0/24
```

## Troubleshooting

```bash
# Test proxy
proxytunnel -p PROXY:8080 -d TARGET:22 -a 127.0.0.1:2222 -v

# Check port
ss -tlnp | grep 2222

# Kill tunnel
pkill proxytunnel
```
