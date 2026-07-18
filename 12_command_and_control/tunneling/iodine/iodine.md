---
title: "iodine - IPv4 over DNS Tunnel"
description: "Tunnel IPv4 data through DNS servers for network access when internet is firewalled"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "iodine"
kali_install: "sudo apt install iodine"
version: "0.7.0"
---

# iodine

## Overview

iodine tunnels IPv4 data through a DNS server. It creates a virtual network interface that routes IP traffic through DNS queries, providing full network connectivity when only DNS resolution is permitted. Unlike other DNS tunneling tools, iodine operates at the IP layer, supporting any protocol that runs over IP.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt install iodine
```

### From Source

```bash
git clone https://github.com/yarrick/iodine.git
cd iodine
make
sudo make install
```

### Verify Installation

```bash
iodine -h
```

## Chapter 2: Basic Usage

### Start Server

```bash
sudo iodined 10.0.0.1 tunnel.example.com -P password
```

**Explanation**: Starts the iodine server. `10.0.0.1` is the IP address for the virtual tunnel interface. `tunnel.example.com` is the topdomain delegated to the server. `-P` sets the authentication password.

### Start Client

```bash
sudo iodine -P password tunnel.example.com
```

**Explanation**: Connects to the iodine server through DNS. The client creates a virtual network interface (typically `dns0`) and configures it with an IP address in the tunnel subnet.

### Verify Connection

```bash
# Check tunnel interface
ip addr show dns0

# Ping through tunnel
ping 10.0.0.1
```

**Explanation**: The `dns0` interface is the virtual tunnel interface. Pings to the tunnel IP confirm the connection is working.

## Chapter 3: Server Configuration

### Full Server Command

```bash
sudo iodined -f -c -P password -l 53 10.0.0.1/24 tunnel.example.com
```

**Explanation**: `-f` runs in foreground, `-c` disables client IP checking, `-P` sets the password, `-l 53` listens on port 53. The `/24` specifies the tunnel subnet mask.

### Daemon Mode

```bash
sudo iodined -P password 10.0.0.1/24 tunnel.example.com
```

**Explanation**: Runs as a daemon by default. Logs to syslog. The `-f` flag is needed to run in the foreground for debugging.

### Port Configuration

```bash
sudo iodined -p 5353 -P password 10.0.0.1/24 tunnel.example.com
```

**Explanation**: The `-p` flag changes the listening port. Use a non-standard port when port 53 is already in use by another DNS server.

## Chapter 4: Client Configuration

### Basic Client

```bash
sudo iodine -P password tunnel.example.com
```

**Explanation**: Connects to the server and creates a tunnel interface. The client automatically configures routing for the tunnel subnet.

### Custom DNS Server

```bash
sudo iodine -r 8.8.8.8 -P password tunnel.example.com
```

**Explanation**: The `-r` flag specifies a relay nameserver. This can be useful when the local DNS resolver cannot reach the tunnel domain.

### Force Record Type

```bash
sudo iodine -T TXT -P password tunnel.example.com
```

**Explanation**: The `-T` flag forces a specific DNS record type (NULL, PRIVATE, TXT, SRV, MX, CNAME, A). Use this when the default auto-detection fails or a specific record type is required.

## Chapter 5: Routing

### Configure Routes

```bash
# Route entire network through tunnel
sudo ip route add 10.0.0.0/24 dev dns0

# Route specific subnet
sudo ip route add 192.168.1.0/24 dev dns0
```

**Explanation**: Routes are configured to direct traffic through the tunnel interface. The server's network is accessible through the tunnel.

### Server-Side Routing

```bash
# Enable IP forwarding on server
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT traffic from tunnel
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

**Explanation**: The server must forward traffic from the tunnel to the internet or internal networks. NAT masquerading hides tunnel clients behind the server's IP address.

## Chapter 6: Performance

### MTU Configuration

```bash
# Check current MTU
ip link show dns0

# Adjust MTU
sudo ip link set dns0 mtu 384
```

**Explanation**: DNS tunneling has lower MTU than standard Ethernet. The default MTU is 1024, but DNS servers may limit packet sizes. Lowering the MTU to 384-512 can improve reliability.

### Fragment Size

```bash
sudo iodine -m 1024 -P password tunnel.example.com
```

**Explanation**: The `-m` flag sets the maximum downstream fragment size. Adjust based on DNS server limitations and network conditions.

### Bandwidth Considerations

iodine provides approximately 10-50 kbps depending on:
- DNS query frequency
- Network latency
- DNS server response time
- Fragment size limitations

**Explanation**: DNS tunneling is inherently slow due to the overhead of encoding data in DNS queries. It is suitable for lightweight C2 channels but not for bulk data transfers.

## Chapter 7: Security

### Authentication

```bash
# Server with password
sudo iodined -P password 10.0.0.1/24 tunnel.example.com

# Client with password
sudo iodine -P password tunnel.example.com
```

**Explanation**: The `-P` flag sets a shared password for authentication. Both server and client must use the same password. Maximum password length is 32 characters.

### Encryption

iodine encrypts all tunnel traffic using the provided password. The encryption uses a combination of hashing and XOR operations. While not cryptographically strong, it provides basic protection against casual inspection.

## Chapter 8: DNS Setup

### Delegate Subdomain

```bash
# In your DNS provider's control panel:
# Add NS record: tunnel.example.com -> ns1.attacker.com
# Add A record: ns1.attacker.com -> ATTACKER_IP
```

**Explanation**: The topdomain must be delegated to an authoritative DNS server you control. This ensures all DNS queries for the tunnel reach your iodine server.

### Verify Delegation

```bash
dig ns tunnel.example.com
dig A ns1.tunnel.example.com
```

**Explanation**: Verify that the NS and A records are correctly configured before starting the tunnel. Incorrect delegation will prevent the tunnel from working.

### Firewall Rules

```bash
# Allow DNS traffic
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT
```

**Explanation**: Ensure the firewall allows DNS traffic (UDP/TCP port 53) to reach the iodine server. This is typically already permitted for DNS servers.

## Resources

- [iodine Homepage](https://code.kryo.se/iodine/)
- [iodine GitHub](https://github.com/yarrick/iodine)
- [Kali Linux iodine Page](https://www.kali.org/tools/iodine/)
- [DNS Tunneling RFC](https://tools.ietf.org/html/rfc1035)
