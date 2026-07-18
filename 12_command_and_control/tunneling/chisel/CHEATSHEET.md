---
title: "chisel CHEATSHEET"
description: "Quick reference for chisel tunneling commands"
---

# chisel CHEATSHEET

## Server

```bash
chisel server -p 8080                              # Basic server
chisel server -p 8080 --reverse                    # Allow reverse tunnels
chisel server -p 8080 --socks5                     # Enable SOCKS5 proxy
chisel server -p 8080 --auth user:pass             # Require authentication
chisel server -p 8080 --authfile users.json        # Auth from file
chisel server -p 443 --tls-domain example.com      # With Let's Encrypt TLS
chisel server -p 443 --tls-key key.pem --tls-cert cert.pem  # Custom TLS
```

## Client

```bash
# Basic connection
chisel client SERVER:8080 R:8081:127.0.0.1:80     # Reverse forward

# Local forward
chisel client SERVER:8080 8081:target:80           # Local forward

# With authentication
chisel client SERVER:8080 --auth user:pass R:8081:target:80

# With fingerprint verification
chisel client --fingerprint FP SERVER:8080 R:8081:target:80

# SOCKS5 proxy
chisel client SERVER:8080 socks                    # SOCKS5 on 1080
chisel client SERVER:8080 5000:socks               # SOCKS5 on 5000

# UDP forwarding
chisel client SERVER:8080 5353/udp:target:53/udp

# Through HTTP proxy
chisel client SERVER:8080 --proxy http://proxy:8080 R:8081:target:80

# HTTPS connection
chisel client https://chisel.example.com R:2222:localhost:22

# With max retries
chisel client SERVER:8080 --max-retry-count 10 R:8081:target:80
```

## Common Use Cases

```bash
# Expose internal web server
chisel server -p 8080 --reverse
chisel client SERVER:8080 R:8080:internal_web:80

# Access internal SSH
chisel server -p 8080 --reverse
chisel client SERVER:8080 R:2222:10.0.0.5:22

# SOCKS proxy for internal network
chisel server -p 8080 --socks5 --reverse
chisel client SERVER:8080 R:socks

# Pivot through compromised host
chisel server -p 8080 --reverse --auth operator:secret
chisel client SERVER:8080 --auth operator:secret R:8080:192.168.1.100:80
```

## Remote Shell with SSH ProxyCommand

```bash
ssh -o ProxyCommand='chisel client SERVER:8080 stdio:%h:%p' user@target_host
```

## Server Users File

```json
{
  "admin:password": [""],
  "user1:pass1": ["^10\\.0\\.0\\.1:.*$"],
  "user2:pass2": ["^R:127\\.0\\.0\\.1:1080$"]
}
```

## Common Flags

### Server Flags

| Flag | Description |
|------|-------------|
| `-p, --port` | Listen port (default: 8080) |
| `--host` | Listen host (default: 0.0.0.0) |
| `--auth USER:PASS` | Single user auth |
| `--authfile FILE` | Users file |
| `--reverse` | Allow reverse tunnels |
| `--socks5` | Enable SOCKS5 proxy |
| `--tls-key FILE` | TLS private key |
| `--tls-cert FILE` | TLS certificate |
| `--tls-domain DOMAIN` | Let's Encrypt domain |
| `--keepalive DURATION` | Keepalive interval (default: 25s) |

### Client Flags

| Flag | Description |
|------|-------------|
| `--fingerprint FP` | Server key fingerprint |
| `--auth USER:PASS` | Authentication |
| `--proxy URL` | HTTP/SOCKS proxy |
| `--header "K:V"` | Custom HTTP header |
| `--keepalive DURATION` | Keepalive interval |
| `--max-retry-count N` | Max reconnect attempts |
| `--tls-ca FILE` | CA certificate |
| `--tls-skip-verify` | Skip TLS verification |
