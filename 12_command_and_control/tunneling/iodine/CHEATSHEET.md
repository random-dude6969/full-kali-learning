---
title: "iodine CHEATSHEET"
description: "Quick reference for iodine DNS tunneling commands"
---

# iodine CHEATSHEET

## Server

```bash
# Basic server
sudo iodined 10.0.0.1/24 tunnel.example.com -P password

# Foreground with debugging
sudo iodined -f -D -c -P password -l 53 10.0.0.1/24 tunnel.example.com

# Custom port
sudo iodined -p 5353 -P password 10.0.0.1/24 tunnel.example.com

# Disable client check
sudo iodined -c -P password 10.0.0.1/24 tunnel.example.com
```

## Client

```bash
# Basic connection
sudo iodine -P password tunnel.example.com

# With relay DNS
sudo iodine -r 8.8.8.8 -P password tunnel.example.com

# Force TXT records
sudo iodine -T TXT -P password tunnel.example.com

# Custom device
sudo iodine -d mytun -P password tunnel.example.com

# Adjust MTU
sudo iodine -m 384 -P password tunnel.example.com
```

## Verify Connection

```bash
# Check tunnel interface
ip addr show dns0

# Ping tunnel endpoint
ping 10.0.0.1

# Check routes
ip route show
```

## Server-Side Routing

```bash
# Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# NAT tunnel traffic
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Forward DNS
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
```

## Client-Side Routing

```bash
# Route specific network through tunnel
sudo ip route add 192.168.1.0/24 dev dns0

# Route all traffic through tunnel
sudo ip route add default dev dns0

# Remove route
sudo ip route del 192.168.1.0/24 dev dns0
```

## Common Flags

| Flag | Description |
|------|-------------|
| `-P PASSWORD` | Authentication password |
| `-p PORT` | Listen/connect port |
| `-r SERVER` | Relay DNS server |
| `-T TYPE` | Force DNS record type |
| `-m SIZE` | Max fragment size |
| `-d DEVICE` | Tunnel device name |
| `-f` | Run in foreground |
| `-c` | Disable client checking |
| `-D` | Debug mode |
| `-v` | Verbose output |

## DNS Record Types

| Type | Description | Recommended |
|------|-------------|-------------|
| NULL | NULL records, low overhead | Yes |
| TXT | Text records, common but monitored | Yes |
| MX | Mail exchange records | No |
| CNAME | Canonical name records | No |
| A | Address records | No |
| SRV | Service records | No |
| PRIVATE | EDNS0 private records | Yes |

## Practical Workflow

```bash
# 1. Set up DNS delegation
# Delegate tunnel.example.com to your authoritative DNS server

# 2. Start server
sudo iodined -f -D -P MyPassword 10.0.0.1/24 tunnel.example.com

# 3. Start client
sudo iodine -P MyPassword tunnel.example.com

# 4. Verify
ping 10.0.0.1

# 5. Route traffic
sudo ip route add 10.10.10.0/24 dev dns0
ssh user@10.10.10.1
```

## Troubleshooting

```bash
# Test DNS resolution
dig tunnel.example.com

# Check if port 53 is available
sudo ss -tlnp | grep :53

# Stop systemd-resolved
sudo systemctl stop systemd-resolved

# Check tunnel interface
ip link show dns0

# Test with lower MTU
sudo iodine -m 256 -P password tunnel.example.com
```

## Bandwidth Tips

- Use NULL or TXT records for best throughput
- Lower MTU for unreliable networks
- Increase fragment size for better performance
- Use relay DNS for better connectivity
- Expect 10-50 kbps throughput
