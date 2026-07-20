---
title: "miredo - Teredo IPv6 Tunneling"
description: "Tunnel IPv6 packets through IPv4 networks using Teredo relay and tunnel broker"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "miredo"
kali_install: "sudo apt install miredo"
version: "1.2.6"
---

# miredo

## Overview

miredo is a Teredo tunneling implementation for Linux. It tunnels IPv6 packets through IPv4 networks using the Teredo protocol, which encapsulates IPv6 packets within UDP datagrams. Teredo is useful for providing IPv6 connectivity to IPv4-only networks and for creating covert channels that appear as normal UDP traffic.

miredo implements the Teredo protocol as defined in RFC 4380. It can operate in three modes: client, relay, and server. In client mode, it connects to a Teredo server to obtain a Teredo address and establish a tunnel. In relay mode, it forwards Teredo packets between Teredo clients and the native IPv6 internet. In server mode, it assigns Teredo addresses to clients and assists with NAT traversal.

The tool is particularly useful for providing IPv6 connectivity in environments where only IPv4 is available. It can also be used for covert communication channels that appear as normal UDP traffic.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install miredo
```

### From Source

```bash
wget https://www.remlab.net/files/miredo/miredo-1.2.6.tar.gz
tar xzf miredo-1.2.6.tar.gz
cd miredo-1.2.6
./configure
make
sudo make install
```

### Verify Installation

```bash
miredo --version
```

### Check Dependencies

```bash
# Check if TUN module is available
lsmod | grep tun

# Load if needed
sudo modprobe tun
```

## Chapter 2: Basic Usage

### Start as Client

```bash
sudo miredo
```

**Explanation**: Starts miredo as a Teredo client, connecting to the default Teredo relay (teredo.remlab.net). The client obtains a Teredo address and creates a `teredo` tunnel interface.

### Start as Relay

```bash
sudo miredo -r
```

**Explanation**: `-r` starts miredo in relay mode. The relay forwards Teredo packets between Teredo clients and the native IPv6 internet.

### Start as Server

```bash
sudo miredo -s
```

**Explanation**: `-s` starts miredo as a Teredo server. The server assigns Teredo addresses to clients and assists with NAT traversal.

### Start with systemd

```bash
sudo systemctl start miredo
sudo systemctl enable miredo
```

**Explanation**: miredo can be managed as a systemd service. The service reads configuration from `/etc/miredo.conf`.

## Chapter 3: Configuration

### Configuration File

```
# /etc/miredo.conf
InterfaceName teredo
ServerAddress 193.219.129.1
TunnelMTU 1280
TunnelIPv6 yes
```

**Explanation**: The configuration file controls the tunnel interface name, server address, MTU size, and IPv6 settings. The default configuration works for most use cases.

### Custom Server

```bash
sudo miredo -c /etc/miredo.conf
```

**Explanation**: The `-c` flag specifies a custom configuration file. Use this to configure a custom Teredo server or tunnel parameters.

### Specify Interface

```
# /etc/miredo.conf
InterfaceName mytun
```

**Explanation**: Changes the tunnel interface name from the default `teredo` to a custom name. Useful when running multiple tunnel instances.

### Configuration Options

```
# /etc/miredo.conf
InterfaceName teredo
ServerAddress 193.219.129.1
TunnelMTU 1280
TunnelIPv6 yes
RequireTunnel no
DiscloseIPv6 no
```

**Explanation**: Various options control tunnel behavior. `RequireTunnel` requires a qualified tunnel (not a relay), `DiscloseIPv6` controls IPv6 address disclosure.

## Chapter 4: Teredo Protocol

### How Teredo Works

Teredo encapsulates IPv6 packets within UDP datagrams to traverse IPv4 NAT devices. The protocol assigns each client a Teredo address derived from the server's IP address and the client's NAT mapping.

**Explanation**: Teredo clients communicate through a Teredo server that helps with NAT traversal. Once a tunnel is established, IPv6 packets are encapsulated in UDP and sent through the IPv4 network.

### Teredo Address Format

```
2001:0000:4136:9060:0000:0000:0000:0000
```

**Explanation**: A Teredo address consists of the Teredo prefix (2001:0000:), the server's IP address, flags, the client's mapped port, and the client's mapped IP address. This encoding allows direct peer-to-peer communication.

### NAT Traversal

```bash
# Check Teredo address
ip -6 addr show teredo

# Check Teredo status
miredo status
```

**Explanation**: The Teredo address indicates successful NAT traversal. The `miredo status` command shows the current tunnel state and connection information.

### NAT Types

Teredo supports three NAT types:
- **Independent**: No NAT mapping required
- **Dependent**: NAT mapping is required
- **Symmetric**: Most restrictive, requires server assistance

**Explanation**: Different NAT types affect how Teredo clients communicate. Symmetric NAT is the most restrictive and may limit peer-to-peer connectivity.

## Chapter 5: Networking

### Verify Tunnel Interface

```bash
ip -6 addr show teredo
ip -6 route show
```

**Explanation**: The `teredo` interface should have an IPv6 address. Routes should include the Teredo prefix and any learned IPv6 routes.

### Ping Through Tunnel

```bash
ping6 2001:4860:4860::8888
```

**Explanation**: Pings a public IPv6 address (Google DNS) through the Teredo tunnel. This verifies that the tunnel is working and IPv6 connectivity is available.

### DNS Resolution

```bash
# Test IPv6 DNS
dig AAAA google.com
host -t AAAA google.com
```

**Explanation**: DNS AAAA records return IPv6 addresses. Testing DNS resolution over IPv6 verifies that the tunnel provides full IPv6 connectivity.

### Route Configuration

```bash
# Add specific route
sudo ip -6 route add 2001:db8::/32 via teredo

# Add default route
sudo ip -6 route add default via teredo
```

**Explanation**: Routes determine which IPv6 traffic goes through the Teredo tunnel. Add routes for specific networks or use a default route for all IPv6 traffic.

## Chapter 6: Security Considerations

### Covert Channel

Teredo traffic appears as normal UDP traffic to network monitors. The encapsulated IPv6 packets are hidden within the UDP payload, providing a potential covert channel.

**Explanation**: While Teredo is designed for IPv6 connectivity, its UDP encapsulation can be used to tunnel arbitrary traffic through networks that permit UDP but block IPv6.

### Detection Methods

- Unusual UDP traffic patterns
- Teredo-specific packet signatures
- Non-standard Teredo server usage
- IPv6-in-IPv4 encapsulation

**Explanation**: Security monitoring tools can detect Teredo traffic through packet signatures and traffic analysis. Using standard Teredo servers reduces detection risk.

### Evasion Techniques

```bash
# Use standard Teredo servers
# /etc/miredo.conf
ServerAddress 193.219.129.1

# Use common port
# Teredo uses UDP port 3544 by default
```

**Explanation**: Using standard Teredo servers and ports makes the traffic look like legitimate IPv6 connectivity.

## Chapter 7: Advanced Configuration

### Custom Teredo Server

```bash
# On your server
sudo miredo -s

# On clients
# /etc/miredo.conf
ServerAddress YOUR_SERVER_IP
```

**Explanation**: Running your own Teredo server provides control over address assignment and reduces reliance on public servers. The server must have public IPv4 and IPv6 connectivity.

### Relay Configuration

```bash
# On relay server
sudo miredo -r

# In miredo.conf
ServerIPv6 2001:0000:4136:9060::
```

**Explanation**: The relay forwards Teredo packets between Teredo clients and the native IPv6 internet. The relay must have both IPv4 and IPv6 connectivity.

### Multiple Interfaces

```bash
# /etc/miredo.conf
InterfaceName teredo1
```

```bash
# /etc/miredo2.conf
InterfaceName teredo2
```

```bash
# Run both
sudo miredo -c /etc/miredo.conf
sudo miredo -c /etc/miredo2.conf
```

**Explanation**: Multiple miredo instances can run simultaneously with different interface names, providing separate tunnels to different Teredo servers.

### Firewall Configuration

```bash
# Allow Teredo traffic (UDP port 3544)
sudo iptables -A INPUT -p udp --dport 3544 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 3544 -j ACCEPT

# Allow IPv6 traffic
sudo ip6tables -A INPUT -i teredo -j ACCEPT
sudo ip6tables -A OUTPUT -o teredo -j ACCEPT
```

**Explanation**: Configure firewall rules to allow Teredo traffic and IPv6 traffic through the tunnel interface.

## Chapter 8: Real-World Scenarios

### IPv6 Connectivity

```bash
# Start miredo client
sudo miredo

# Verify IPv6 address
ip -6 addr show teredo

# Test IPv6 connectivity
ping6 2001:4860:4860::8888
curl -6 http://ipv6.google.com
```

**Explanation**: Use miredo to provide IPv6 connectivity to IPv4-only networks. This is useful for testing IPv6 applications or accessing IPv6-only services.

### Covert Communication

```bash
# Server
sudo miredo -s

# Client
# /etc/miredo.conf
ServerAddress SERVER_IP
```

**Explanation**: Use Teredo for covert communication channels that appear as normal UDP traffic. The encapsulated IPv6 packets can carry arbitrary data.

### Network Bridging

```bash
# Bridge IPv4 and IPv6 networks
sudo miredo
sudo ip6tables -t nat -A POSTROUTING -o teredo -j MASQUERADE
sudo sysctl -w net.ipv6.conf.all.forwarding=1
```

**Explanation**: Bridge IPv4 and IPv6 networks using miredo. This provides IPv6 connectivity to IPv4-only hosts on the network.

## Chapter 9: Detection and Defense

### Network Signatures

- UDP traffic on port 3544
- Teredo-specific packet headers
- IPv6-in-IPv4 encapsulation
- Unusual DNS AAAA record queries

### IDS/IPS Detection

```
# Suricata rule for Teredo
alert udp any any -> any 3544 (msg:"Teredo Tunnel Detected"; \
  content:"|00|"; \
  content:"|01|"; \
  threshold:type both,track by_src, count 10, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor UDP traffic
tcpdump -i eth0 udp port 3544 -w teredo.pcap

# Analyze Teredo packets
tshark -r teredo.pcap -Y "udp.port == 3544"

# Check IPv6 routes
ip -6 route show | grep teredo
```

### Defense Recommendations

1. Monitor for UDP traffic on port 3544
2. Block Teredo at network perimeter
3. Implement IPv6 firewall rules
4. Monitor for IPv6-in-IPv4 encapsulation
5. Use DNS64/NAT64 instead of Teredo

## Chapter 10: Troubleshooting

### Connection Issues

```bash
# Check if teredo interface exists
ip link show teredo

# Check Teredo status
miredo status

# View logs
journalctl -u miredo

# Restart service
sudo systemctl restart miredo
```

### Firewall Issues

```bash
# Allow Teredo traffic
sudo iptables -A INPUT -p udp --dport 3544 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 3544 -j ACCEPT

# Allow IPv6 traffic
sudo ip6tables -A INPUT -i teredo -j ACCEPT
sudo ip6tables -A OUTPUT -o teredo -j ACCEPT
```

### Performance Issues

```bash
# Check MTU settings
ip link show teredo | grep mtu

# Adjust MTU if needed
sudo ip link set teredo mtu 1280

# Check for packet loss
ping6 -c 100 2001:4860:4860::8888
```

### NAT Traversal Issues

```bash
# Check NAT type
miredo status

# Try different server
# /etc/miredo.conf
ServerAddress 66.218.86.248
```

### DNS Issues

```bash
# Test DNS resolution
dig AAAA google.com

# Check DNS configuration
cat /etc/resolv.conf

# Add IPv6 DNS server
echo "nameserver 2001:4860:4860::8888" | sudo tee -a /etc/resolv.conf
```

## Resources

- [miredo Homepage](https://www.remlab.net/miredo/)
- [miredo man page](https://www.remlab.net/miredo/miredo.8.html)
- [RFC 4380 - Teredo](https://tools.ietf.org/html/rfc4380)
- [Kali Linux miredo Page](https://www.kali.org/tools/miredo/)
- [Teredo Tunneling](https://en.wikipedia.org/wiki/Teredo_tunneling)
- [MITRE ATT&CK - Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [MITRE ATT&CK - Tunnel IPv6](https://attack.mitre.org/techniques/T1572/)
- [IPv6 Security Best Practices](https://www.ietf.org/rfc/rfc4890.txt)

---

## Chapter 11: Comparison with Other Tunneling Tools

### Miredo vs Other IPv6 Tunneling

| Feature | Miredo | 6in4 | AYIYA | WireGuard |
|---|---|---|---|---|
| Protocol | UDP | IPv4 | UDP | UDP |
| NAT Traversal | Yes | No | Yes | Yes |
| Privileges | Root | Root | Root | Root |
| Stealth | Moderate | Low | Moderate | High |
| Setup Complexity | Low | Medium | High | Medium |

### When to Use Miredo

- Providing IPv6 to IPv4-only networks
- Creating covert channels using UDP encapsulation
- Legacy application support over IPv6
- Network penetration testing requiring IPv6
- Covert communications over restrictive networks

### When to Use Alternatives

- **6in4**: When static tunnel endpoints are available
- **AYIYA**: When TCP-like reliability is needed over UDP
- **WireGuard**: When high-performance encryption is required
- **OpenVPN**: When full VPN functionality is needed

### Teredo Address Structure

```
2001:0000:4136:9060:0000:0000:0000:0000
|______| |____| |____________________|
Prefix   Server   Client Mapping
         IP       (Port + IP)
```
