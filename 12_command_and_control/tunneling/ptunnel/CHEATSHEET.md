---
title: "ptunnel CHEATSHEET"
description: "Quick reference for ptunnel ICMP tunneling commands"
---

# ptunnel CHEATSHEET

## Basic Usage

```bash
# Start proxy (requires root)
sudo ptunnel

# Connect client
ptunnel -p PROXY_IP -lp 8000 -da TARGET_IP -dp 22

# SSH through tunnel
ssh -p 8000 user@127.0.0.1
```

## Common Configurations

```bash
# SSH tunnel
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22

# HTTP tunnel
ptunnel -p PROXY_IP -lp 8080 -da TARGET:80 -dp 80

# RDP tunnel
ptunnel -p PROXY_IP -lp 3389 -da TARGET:3389 -dp 3389

# With password
sudo ptunnel -pass MyPassword
ptunnel -p PROXY_IP -pass MyPassword -lp 8000 -da TARGET:22 -dp 22

# Verbose
ptunnel -p PROXY_IP -lp 8000 -da TARGET:22 -dp 22 -v
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-p PROXY` | Proxy address |
| `-lp PORT` | Local listen port |
| `-da ADDR` | Destination address |
| `-dp PORT` | Destination port |
| `-pass PASS` | Password |
| `-v` | Verbose mode |
| `-mtu SIZE` | MTU size |
| `-id ID` | ICMP identifier |
| `-daemon` | Run as daemon |

## Practical Workflow

```bash
# 1. Start proxy on relay server
sudo ptunnel -pass SecretPass

# 2. Connect client
ptunnel -p RELAY_IP -pass SecretPass -lp 8000 -da TARGET:22 -dp 22

# 3. SSH through tunnel
ssh -p 8000 user@127.0.0.1

# 4. Create SOCKS proxy
ssh -D 1080 -p 8000 user@127.0.0.1

# 5. Scan through proxy
proxychains4 nmap -sT 10.10.10.0/24
```

## Troubleshooting

```bash
# Test ICMP connectivity
ping PROXY_IP

# Check local port
ss -tlnp | grep 8000

# Kill ptunnel
pkill ptunnel

# Check logs
journalctl -u ptunnel
```

## Limitations

- ICMP only (no UDP/TCP direct)
- Low bandwidth (10-100 kbps)
- Requires root on proxy side
- May be blocked by ICMP filtering
- Not suitable for bulk data transfer
