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

iodine works by encoding IP packets into DNS queries and responses. The client sends DNS queries to a server it controls, and the server responds with DNS responses containing the tunneled data. This creates a bidirectional tunnel that can carry any IP traffic.

The tool is particularly useful in highly restricted network environments where only DNS traffic is allowed. It can bypass firewalls that only permit DNS queries to external servers. iodine supports authentication, encryption, and multiple DNS record types.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
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
iodined -h
```

### Check Kernel Module

```bash
# Check if TUN module is loaded
lsmod | grep tun

# Load if needed
sudo modprobe tun
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

# Check routing
ip route show
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

### Network Configuration

```bash
# Specify tunnel network and mask
sudo iodined -P password 10.0.0.1/24 tunnel.example.com

# Use larger subnet
sudo iodined -P password 10.0.0.0/16 tunnel.example.com
```

**Explanation**: The tunnel subnet defines the IP addresses used by the tunnel. Choose a subnet that doesn't conflict with existing networks.

### Server Options

```bash
# Run in foreground with debug
sudo iodined -f -d2 -P password 10.0.0.1/24 tunnel.example.com

# Disable client checking
sudo iodined -c -P password 10.0.0.1/24 tunnel.example.com

# Set maximum packet size
sudo iodined -m 1280 -P password 10.0.0.1/24 tunnel.example.com
```

**Explanation**: Various options control server behavior. The `-d` flag sets debug level, `-c` disables client IP checking, and `-m` sets maximum downstream fragment size.

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

### Client Options

```bash
# Run in foreground with debug
sudo iodine -f -d2 -P password tunnel.example.com

# Set MTU
sudo iodine -M 512 -P password tunnel.example.com

# Use specific DNS server
sudo iodine -r 8.8.8.8 -P password tunnel.example.com
```

**Explanation**: Various options control client behavior. The `-M` flag sets the MTU, `-r` specifies a DNS server, and `-d` sets debug level.

## Chapter 5: Routing

### Configure Routes

```bash
# Route entire network through tunnel
sudo ip route add 10.0.0.0/24 dev dns0

# Route specific subnet
sudo ip route add 192.168.1.0/24 dev dns0

# Route specific host
sudo ip route add 192.168.1.100/32 dev dns0
```

**Explanation**: Routes are configured to direct traffic through the tunnel interface. The server's network is accessible through the tunnel.

### Server-Side Routing

```bash
# Enable IP forwarding on server
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT traffic from tunnel
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Allow forwarded traffic
sudo iptables -A FORWARD -s 10.0.0.0/24 -j ACCEPT
sudo iptables -A FORWARD -d 10.0.0.0/24 -j ACCEPT
```

**Explanation**: The server must forward traffic from the tunnel to the internet or internal networks. NAT masquerading hides tunnel clients behind the server's IP address.

### Persistent Routing

```bash
# Add to /etc/network/interfaces
auto dns0
iface dns0 inet static
    address 10.0.0.2
    netmask 255.255.255.0

# Add route script
sudo ip route add 10.0.0.0/24 dev dns0
```

**Explanation**: Configure persistent routing for the tunnel interface. This ensures routes are available after reboots.

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
# Set maximum downstream fragment size
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

### Performance Optimization

```bash
# Use NULL record type (fastest)
sudo iodine -T NULL -P password tunnel.example.com

# Increase fragment size
sudo iodine -m 2048 -P password tunnel.example.com

# Use local DNS server
sudo iodine -r 192.168.1.1 -P password tunnel.example.com
```

**Explanation**: NULL record type provides the best performance as it has minimal overhead. Larger fragment sizes reduce the number of queries needed.

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

### Password Best Practices

```bash
# Use strong passwords
sudo iodined -P 'Kj#9x!mP2$wL' 10.0.0.1/24 tunnel.example.com

# Change passwords regularly
# Update both server and client configurations
```

**Explanation**: Use strong, unique passwords for iodine tunnels. Change passwords regularly to limit exposure if credentials are compromised.

### Additional Security Measures

```bash
# Restrict client IPs on server
sudo iodined -c -P password 10.0.0.1/24 tunnel.example.com

# Use firewall rules
sudo iptables -A INPUT -p udp --dport 53 -s TRUSTED_IP -j ACCEPT
sudo iptables -A INPUT -p udp --dport 53 -j DROP
```

**Explanation**: Combine iodine's authentication with firewall rules for defense in depth. Restrict which IPs can connect to the iodine server.

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
# Check NS record
dig ns tunnel.example.com

# Check A record
dig A ns1.tunnel.example.com

# Test DNS resolution
dig @ns1.tunnel.example.com test.tunnel.example.com
```

**Explanation**: Verify that the NS and A records are correctly configured before starting the tunnel. Incorrect delegation will prevent the tunnel from working.

### Firewall Rules

```bash
# Allow DNS traffic
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Allow from specific source
sudo iptables -A INPUT -p udp --dport 53 -s 0.0.0.0/0 -j ACCEPT
```

**Explanation**: Ensure the firewall allows DNS traffic (UDP/TCP port 53) to reach the iodine server. This is typically already permitted for DNS servers.

### DNS Record Types

```bash
# NULL record (fastest, requires authoritative server)
# Use when you control the authoritative DNS server

# TXT record (widely supported)
# Works through most DNS resolvers

# SRV record (less common)
# May be blocked by some firewalls

# MX record (unusual)
# May raise suspicion

# CNAME record (limited)
# May have encoding limitations
```

**Explanation**: Different DNS record types have different performance characteristics and compatibility. NULL is fastest but requires authoritative server control. TXT is most compatible.

## Chapter 9: Real-World Scenarios

### Bypass Firewall

```bash
# Server (outside firewall)
sudo iodined -f -c -P password 10.0.0.1/24 tunnel.example.com

# Client (inside firewall)
sudo iodine -f -d2 -P password tunnel.example.com

# Test connectivity
ping 10.0.0.1
curl http://10.0.0.2
```

**Explanation**: Use iodine to bypass restrictive firewalls that only allow DNS traffic. The tunnel provides full network access through DNS queries.

### Remote Access

```bash
# Server
sudo iodined -P password 10.0.0.1/24 tunnel.example.com

# Client
sudo iodine -P password tunnel.example.com

# SSH through tunnel
ssh user@10.0.0.2
```

**Explanation**: Use iodine for remote access when direct SSH connections are blocked. The DNS tunnel provides a path for SSH traffic.

### Emergency Connectivity

```bash
# When all other protocols are blocked
sudo iodine -P password tunnel.example.com

# Access internal resources
ssh user@192.168.1.100
curl http://192.168.1.200
```

**Explanation**: iodine provides emergency connectivity when only DNS is available. This can be critical for incident response and emergency access.

## Chapter 10: Detection and Defense

### Network Signatures

- High volume of DNS queries to a single domain
- Unusual DNS record types (NULL, TXT with binary data)
- Consistent DNS query sizes
- Long-lived DNS sessions

### IDS/IPS Detection

```
# Suricata rule for iodine
alert udp any any -> any 53 (msg:"DNS Tunnel Detected"; \
  content:"tunnel.example.com"; \
  threshold:type both,track by_src, count 100, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor DNS query volume
tcpdump -i eth0 port 53 -w dns.pcap

# Analyze DNS queries
tshark -r dns.pcap -Y "dns.qry.name contains tunnel.example.com"

# Check for unusual DNS patterns
grep -E "tunnel\.example\.com" /var/log/syslog
```

### Defense Recommendations

1. Monitor for high-volume DNS queries to single domains
2. Implement DNS response policy zones (RPZ)
3. Use DNS security extensions (DNSSEC)
4. Monitor for unusual DNS record types
5. Implement network segmentation

## Chapter 11: Troubleshooting

### Connection Issues

```bash
# Check DNS resolution
dig ns tunnel.example.com

# Test with different record type
sudo iodine -T TXT -P password tunnel.example.com

# Check firewall rules
sudo iptables -L -n | grep 53
```

### Performance Issues

```bash
# Lower MTU
sudo ip link set dns0 mtu 384

# Use NULL record type
sudo iodine -T NULL -P password tunnel.example.com

# Use local DNS server
sudo iodine -r 192.168.1.1 -P password tunnel.example.com
```

### Authentication Errors

```bash
# Verify password matches
# Check server and client passwords are identical

# Check password length (max 32 characters)
echo -n "password" | wc -c
```

### Routing Issues

```bash
# Check routes
ip route show

# Add missing route
sudo ip route add 10.0.0.0/24 dev dns0

# Check IP forwarding
cat /proc/sys/net/ipv4/ip_forward
```

### Kernel Module Issues

```bash
# Check TUN module
lsmod | grep tun

# Load if needed
sudo modprobe tun

# Check device permissions
ls -la /dev/net/tun
```

## Resources

- [iodine Homepage](https://code.kryo.se/iodine/)
- [iodine GitHub](https://github.com/yarrick/iodine)
- [Kali Linux iodine Page](https://www.kali.org/tools/iodine/)
- [DNS Tunneling RFC 1035](https://tools.ietf.org/html/rfc1035)
- [iodine man page](https://linux.die.net/man/8/iodine)
