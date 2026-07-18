---
title: "miredo CHEATSHEET"
description: "Quick reference for miredo Teredo tunneling commands"
---

# miredo CHEATSHEET

## Basic Operations

```bash
sudo miredo                                   # Start as client
sudo miredo -r                                # Start as relay
sudo miredo -s                                # Start as server
sudo miredo -c /etc/miredo.conf               # Custom config
miredo status                                 # Check status
```

## Configuration

```
# /etc/miredo.conf
InterfaceName teredo
ServerAddress 193.219.129.1
TunnelMTU 1280
TunnelIPv6 yes
```

## Verify Connection

```bash
# Check tunnel interface
ip -6 addr show teredo

# Check routes
ip -6 route show

# Ping through tunnel
ping6 2001:4860:4860::8888

# Check status
miredo status
```

## Firewall Rules

```bash
# Allow Teredo traffic
sudo iptables -A INPUT -p udp --dport 3544 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 3544 -j ACCEPT
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-c FILE` | Configuration file |
| `-r` | Relay mode |
| `-s` | Server mode |
| `-d` | Debug mode |
| `-f` | Foreground |
| `-q` | Quiet mode |

## Service Management

```bash
sudo systemctl start miredo                   # Start service
sudo systemctl stop miredo                    # Stop service
sudo systemctl enable miredo                  # Enable on boot
sudo systemctl status miredo                  # Check status
sudo journalctl -u miredo                     # View logs
```

## Troubleshooting

```bash
# Check interface
ip link show teredo

# Check if running
ps aux | grep miredo

# Restart service
sudo systemctl restart miredo

# Check logs
sudo journalctl -u miredo -f
```
