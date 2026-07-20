---
title: "sshuttle - Transparent VPN over SSH"
description: "Transparent proxy/VPN-like tool that tunnels traffic through SSH without requiring a VPN client on the remote server"
category: "tunneling"
mitre_tactic: command_and_control
mitre_tactic_id: TA0011
package: "sshuttle"
kali_install: "sudo apt install sshuttle"
version: "1.1.2"
github: "https://github.com/sshuttle/sshuttle"
---

# sshuttle

## Overview

sshuttle creates a transparent proxy server that routes traffic through an SSH tunnel. Unlike traditional VPNs, sshuttle does not require installation on the remote server — only Python is needed. It uses iptables to capture and forward traffic through the SSH connection, providing a simple but effective VPN-like solution for pivoting through compromised hosts.

sshuttle works by creating a local listener that captures traffic using iptables NAT rules. The captured traffic is then forwarded through an SSH tunnel to the remote server, where it is routed to the final destination. This approach provides transparent routing without requiring VPN clients on either end.

The tool is particularly useful for penetration testing and network administration, where you need to access internal networks through a compromised or jump host. sshuttle supports subnet routing, DNS tunneling, and multiple authentication methods.

## Chapter 1: Installation

### Kali Linux

```bash
sudo apt update
sudo apt install sshuttle
```

### From pip

```bash
pip install sshuttle
pip3 install sshuttle
```

### From Source

```bash
git clone https://github.com/sshuttle/sshuttle.git
cd sshuttle
sudo python3 setup.py install
```

### Verify Installation

```bash
sshuttle --version
```

### Check Dependencies

```bash
# sshuttle requires Python and iptables
python3 --version
iptables --version
```

## Chapter 2: Basic Usage

### Route Subnet Through SSH

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: `-r` specifies the remote SSH server, and the subnet defines which traffic to route through the tunnel. All traffic to 10.0.0.0/8 is routed through the SSH connection.

### Route All Traffic

```bash
sshuttle -r user@attacker_ip 0/0
```

**Explanation**: Routes all traffic (0/0) through the SSH tunnel. This creates a full VPN, redirecting all network traffic through the remote server.

### Auto-Detect Subnets

```bash
sshuttle -r user@attacker_ip --auto-nets
```

**Explanation**: The `--auto-nets` option automatically detects which subnets are accessible through the remote server and routes traffic accordingly.

### Verbose Output

```bash
sshuttle -v -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The `-v` flag enables verbose output, showing connection details and routing information.

## Chapter 3: Subnet Routing

### Multiple Subnets

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 192.168.0.0/16 172.16.0.0/12
```

**Explanation**: Multiple subnets can be specified, each separated by spaces. All traffic to these subnets is routed through the SSH tunnel.

### Exclude Subnets

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -x 10.0.1.0/24
```

**Explanation**: The `-x` flag excludes specific subnets from the tunnel. Traffic to 10.0.1.0/24 remains on the local network while other 10.x traffic goes through the tunnel.

### Subnet File

```bash
sshuttle -r user@attacker_ip -s /path/to/subnets.txt
```

**Explanation**: The `-s` flag specifies a file containing subnets to route. The file should list one subnet per line, useful for managing large numbers of subnets.

### Subnet File Format

```
# subnets.txt
10.0.0.0/8
192.168.0.0/16
172.16.0.0/12
# This is a comment
```

**Explanation**: The subnet file can contain comments (lines starting with #) and blank lines. Each subnet should be on its own line.

### Specific Host Routing

```bash
sshuttle -r user@attacker_ip 10.0.0.100/32
```

**Explanation**: Route traffic to a specific host using /32 CIDR notation. This is useful for accessing a single host on an internal network.

## Chapter 4: DNS Tunneling

### Forward DNS Through Tunnel

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns
```

**Explanation**: The `--dns` option captures local DNS queries and forwards them to the remote DNS server. This prevents DNS leaks that could reveal target hostnames.

### Custom DNS Server

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --dns --to-ns 8.8.8.8
```

**Explanation**: The `--to-ns` option specifies which DNS server to use for forwarded queries. This allows using a specific DNS server on the remote network.

### DNS Leak Prevention

```bash
# Verify DNS is not leaking
dig @8.8.8.8 myip.opendns.com

# Check DNS resolution
nslookup internal-server.local
```

**Explanation**: After setting up sshuttle with DNS forwarding, verify that DNS queries are going through the tunnel and not leaking to local DNS servers.

## Chapter 5: Authentication

### SSH Key Authentication

```bash
sshuttle -r user@attacker_ip -e "ssh -i /path/to/key" 10.0.0.0/8
```

**Explanation**: The `-e` flag specifies a custom SSH command. This allows using SSH key authentication, custom ports, or other SSH options.

### SSH Config

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: sshuttle uses the standard SSH client, which reads `~/.ssh/config`. Configure SSH options there for key files, proxy commands, and other settings.

### Custom SSH Options

```bash
sshuttle -r user@attacker_ip -e "ssh -p 2222 -i key -o StrictHostKeyChecking=no" 10.0.0.0/8
```

**Explanation**: Multiple SSH options can be combined using the `-e` flag. This allows customizing the SSH connection for specific requirements.

### Password Authentication

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: By default, sshuttle uses SSH key authentication. If keys are not configured, sshuttle will prompt for the SSH password.

## Chapter 6: Connection Methods

### NAT Method

```bash
sshuttle --method nat -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The NAT method uses iptables NAT rules to redirect traffic. This is the default method on Linux and provides transparent routing.

### TProxy Method

```bash
sshuttle --method tproxy -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The TProxy method uses iptables TPROXY for transparent proxying. This method supports UDP traffic and may be more compatible with certain network configurations.

### PF Method (BSD)

```bash
sshuttle --method pf -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The PF method uses BSD's packet filter for transparent routing. This is used on macOS and BSD systems.

### Auto Method

```bash
sshuttle --method auto -r user@attacker_ip 10.0.0.0/8
```

**Explanation**: The auto method automatically selects the best available method based on the operating system and available tools.

## Chapter 7: Advanced Features

### Daemon Mode

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -D
```

**Explanation**: The `-D` flag runs sshuttle as a daemon in the background. Useful for persistent tunnels that should survive terminal disconnection.

### Hostname Scanning

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --auto-hosts
```

**Explanation**: The `--auto-hosts` option continuously scans for remote hostnames and updates the local `/etc/hosts` file. This provides hostname resolution for internal services.

### Latency Control

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 --no-latency-control
```

**Explanation**: The `--no-latency-control` option sacrifices latency for improved bandwidth. Use this for bulk data transfers where latency is not critical.

### SSH Compression

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -e "ssh -C"
```

**Explanation**: SSH compression can improve throughput for text-based traffic. The `-C` flag enables compression in the SSH connection.

### Connection Sharing

```bash
sshuttle -r user@attacker_ip 10.0.0.0/8 -e "ssh -M -S /tmp/ssh-socket"
```

**Explanation**: SSH connection sharing allows multiple sshuttle instances to use the same SSH connection. This reduces connection overhead and improves performance.

## Chapter 8: Practical Scenarios

### Internal Network Access

```bash
# SSH to pivot host
ssh -D 1080 user@pivot_host

# Or use sshuttle directly
sshuttle -r user@pivot_host 10.10.10.0/24

# Access internal services
ssh user@10.10.10.5
curl http://10.10.10.10
nmap -sT 10.10.10.0/24
```

**Explanation**: sshuttle provides transparent access to internal networks through a compromised host. Tools work normally without proxychains configuration.

### Multiple Pivots

```bash
# First pivot
sshuttle -r user@external 10.10.10.0/24

# Second pivot through first
sshuttle -r user@10.10.10.5 172.16.0.0/16
```

**Explanation**: sshuttle can chain through multiple pivot points. Each sshuttle instance adds a route to the next network segment.

### Daemon with Auto-Connect

```bash
# Start daemon
sshuttle -r user@attacker_ip 10.0.0.0/8 -D --pidfile /tmp/sshuttle.pid

# Check if running
cat /tmp/sshuttle.pid
ps aux | grep sshuttle

# Stop daemon
kill $(cat /tmp/sshuttle.pid)
```

**Explanation**: Running sshuttle as a daemon with a PID file allows monitoring and management of the tunnel process.

### Web Application Testing

```bash
# Route to internal web application
sshuttle -r user@pivot_host 10.10.10.0/24

# Access web application
curl http://10.10.10.100
nikto -h 10.10.10.100
```

**Explanation**: sshuttle provides transparent access to internal web applications for security testing.

## Chapter 9: Detection and Defense

### Network Signatures

- SSH connections to external hosts
- Unusual iptables rules
- High volume of traffic through SSH

### IDS/IPS Detection

```
# Suricata rule for sshuttle
alert tcp any any -> any 22 (msg:"SSH Tunnel Detected"; \
  flow:to_server,established; \
  content:"SSH-"; \
  threshold:type both,track by_src, count 1, seconds 60; \
  sid:1000001; rev:1;)
```

### Log Analysis

```bash
# Check for iptables rules
iptables -t nat -L -n

# Monitor SSH connections
ss -tnp | grep :22

# Review system logs
grep sshuttle /var/log/syslog
```

### Defense Recommendations

1. Monitor for unusual SSH connections
2. Implement egress filtering
3. Use network segmentation
4. Monitor for iptables rule changes
5. Implement application-layer firewalls

## Chapter 10: Troubleshooting

### Connection Refused

```bash
# Check SSH connectivity
ssh user@attacker_ip

# Check firewall rules
iptables -L -n

# Test with verbose output
sshuttle -v -r user@attacker_ip 10.0.0.0/8
```

### Routing Issues

```bash
# Check routes
ip route show

# Verify subnet routing
traceroute 10.0.0.1

# Check iptables rules
iptables -t nat -L -n -v
```

### DNS Issues

```bash
# Test DNS resolution
dig internal-server.local

# Check DNS forwarding
sshuttle -r user@attacker_ip --dns 10.0.0.0/8

# Verify DNS is going through tunnel
tcpdump -i eth0 port 53
```

### Performance Issues

```bash
# Enable compression
sshuttle -r user@attacker_ip -e "ssh -C" 10.0.0.0/8

# Disable latency control
sshuttle --no-latency-control -r user@attacker_ip 10.0.0.0/8

# Use different method
sshuttle --method tproxy -r user@attacker_ip 10.0.0.0/8
```

### Permission Issues

```bash
# sshuttle requires root for iptables
sudo sshuttle -r user@attacker_ip 10.0.0.0/8

# Or use capabilities
sudo setcap cap_net_admin+ep /usr/bin/sshuttle
```

## Chapter 9: Command Reference

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-r, --remote` | Remote SSH server | none (required) |
| `-e, --ssh-cmd` | Custom SSH command | ssh |
| `-l, --listen` | Local listen address | 127.0.0.1 |
| `--auto-nets` | Auto-detect subnets | disabled |
| `--auto-hosts` | Auto-detect hostnames | disabled |
| `-x, --exclude` | Exclude subnets | none |
| `-s, --subnets` | Subnet file | none |
| `--dns` | Forward DNS queries | disabled |
| `--to-ns` | DNS server for forwarding | system default |
| `-D, --daemon` | Run as daemon | foreground |
| `--method` | NAT method | auto |
| `--no-latency-control` | Disable latency control | enabled |
| `--pidfile` | PID file path | none |
| `-v, --verbose` | Verbose output | quiet |
| `--ns-tls` | Use TLS for DNS | enabled |
| `--no-nfqueue` | Disable NFQUEUE | disabled |

### NAT Methods

| Method | Description | Platform |
|--------|-------------|----------|
| `nat` | iptables NAT | Linux |
| `tproxy` | iptables TPROXY | Linux |
| `pf` | Packet filter | BSD/macOS |
| `ipfw` | IP firewall | macOS |
| `nft` | nftables | Linux |

### Configuration File

```bash
# ~/.ssh/config
Host internal-*
    ProxyJump user@pivot-host
    IdentityFile ~/.ssh/id_rsa_internal
    User admin

# sshuttle uses standard SSH config
sshuttle -r user@pivot-host 10.0.0.0/8
```

**Explanation**: sshuttle leverages SSH client configuration. Configure SSH options in `~/.ssh/config` for key files, proxy commands, and connection settings.

### Environment Variables

```bash
export SSHUTTLE_IPTABLES_ARGS="--dport 53 -j REDIRECT --to-port 12300"
export SSHUTTLE_DEBUG=1
export SSHUTTLE.sulake=1
```

**Explanation**: Environment variables can modify sshuttle's behavior. `SSHUTTLE_DEBUG` enables debug output.

## Chapter 10: Real-World Scenarios

### Scenario 1: Corporate Network Pivot

```bash
# First, establish SSH access to pivot host
ssh -D 1080 user@pivot-corp

# Or use sshuttle for transparent routing
sshuttle -r user@pivot-corp 10.10.0.0/16 --dns

# Access internal services transparently
curl http://intranet.corp.local
ssh admin@dc01.corp.local
nmap -sT 10.10.1.0/24
```

**Explanation**: sshuttle provides transparent access to the corporate network through a compromised host. All tools work without proxychains configuration.

### Scenario 2: Multi-Hop Pivoting

```bash
# Hop 1: Internet to DMZ
sshuttle -r user@dmz-host 10.10.10.0/24

# Hop 2: DMZ to Internal
sshuttle -r user@10.10.10.5 192.168.0.0/16

# Hop 3: Internal to Lab
sshuttle -r user@192.168.1.100 172.16.0.0/12

# Now access lab network
ssh admin@172.16.1.50
```

**Explanation**: sshuttle chains through multiple pivot points, each adding routes to deeper network segments.

### Scenario 3: Cloud Environment Access

```bash
# AWS VPC
sshuttle -r ec2-user@bastion 10.0.0.0/16 --dns

# Azure VNet
sshuttle -r azureuser@jumpbox 10.1.0.0/16 --dns

# GCP VPC
sshuttle -r gcpuser@nativemsg 10.128.0.0/16 --dns
```

**Explanation**: sshuttle provides access to cloud VPCs through bastion hosts, routing traffic through the SSH connection.

### Scenario 4: Restricted Environment

```bash
# When only outbound SSH is allowed
sshuttle -r user@external-vps 10.0.0.0/8 --dns

# Access internal resources
curl http://internal-app.corp.local:8080
ssh admin@internal-db.corp.local

# DNS resolution through tunnel
nslookup internal.corp.local
```

**Explanation**: sshuttle works in restricted environments where only outbound SSH is permitted. The `--dns` option routes DNS queries through the tunnel.

### Scenario 5: Incident Response

```bash
# Quick access to compromised network
sshuttle -r user@investigator-vps 192.168.0.0/16 --dns --auto-nets

# Deploy forensics tools
scp -P 22 tools.tgz user@192.168.1.50:/tmp/

# Collect evidence
ssh user@192.168.1.50 "dd if=/dev/sda1 bs=4M | gzip" > evidence.img.gz
```

**Explanation**: sshuttle provides rapid access to compromised networks for incident response teams, enabling forensic collection and analysis.

## Chapter 11: Detection and Defense

### Network Detection

```bash
# Monitor for SSH connections to unusual IPs
ss -tnp | grep ssh
netstat -tnp | grep ssh

# Check for sshuttle processes
ps aux | grep sshuttle

# Monitor iptables rules for sshuttle
iptables -t nat -L -n | grep sshuttle
```

### Host-Based Detection

```bash
# Check for sshuttle installation
which sshuttle
find / -name "sshuttle" 2>/dev/null

# Monitor Python processes (sshuttle is Python)
ps aux | grep python | grep sshuttle

# Check for iptables modifications
iptables -t nat -L -v
```

### IDS/IPS Signatures

```bash
# Suricata rule for SSH tunneling
alert tcp $HOME_NET any -> $EXTERNAL_NET 22 (
    msg:"Potential SSH Tunnel - Long Session";
    flow:to_server,established;
    dgt:86400;
    threshold:type both,track by_src, count 1, seconds 3600;
    sid:2024001; rev:1;
)
```

### Firewall Rules

```bash
# Limit SSH connections
iptables -A OUTPUT -p tcp --dport 22 -m connlimit --connlimit-above 5 -j DROP

# Block SSH to external IPs
iptables -A OUTPUT -p tcp --dport 22 -d ! 10.0.0.0/8 -j DROP

# Monitor outbound SSH
iptables -A OUTPUT -p tcp --dport 22 -j LOG --log-prefix "SSH_OUTBOUND: "
```

### Defense Recommendations

1. Monitor for long-lived SSH sessions
2. Implement egress filtering for SSH
3. Use network segmentation
4. Monitor for sshuttle/VPN processes
5. Log all SSH connections

## Chapter 12: Troubleshooting

### Connection Issues

```bash
# Test SSH connectivity first
ssh user@pivot-host

# Check if sshuttle is installed on remote
ssh user@pivot-host "which sshuttle"

# Install sshuttle on remote if needed
ssh user@pivot-host "pip install sshuttle"

# Use verbose mode for debugging
sshuttle -v -r user@pivot-host 10.0.0.0/8
```

### DNS Leaks

```bash
# Force DNS through tunnel
sshuttle --dns -r user@pivot-host 10.0.0.0/8

# Test DNS resolution
nslookup internal.corp.local
dig internal.corp.local

# Check for DNS leaks
cat /etc/resolv.conf
```

### Performance Issues

```bash
# Disable latency control for speed
sshuttle --no-latency-control -r user@pivot-host 10.0.0.0/8

# Use TPROXY method for better performance
sshuttle --method tproxy -r user@pivot-host 10.0.0.0/8

# Reduce logging overhead
sshuttle -q -r user@pivot-host 10.0.0.0/8
```

### Daemon Mode

```bash
# Start as daemon
sshuttle -D -r user@pivot-host 10.0.0.0/8 --pidfile /tmp/sshuttle.pid

# Check if running
cat /tmp/sshuttle.pid
ps aux | grep sshuttle

# Stop daemon
kill $(cat /tmp/sshuttle.pid)
```

### Permission Issues

```bash
# sshuttle requires root for iptables
sudo sshuttle -r user@pivot-host 10.0.0.0/8

# Or use setuid (not recommended)
sudo chmod u+s /usr/bin/sshuttle

# Check iptables permissions
sudo iptables -t nat -L -n
```

## Chapter 13: Comparison with Alternatives

### sshuttle vs OpenVPN

| Feature | sshuttle | OpenVPN |
|---------|----------|---------|
| Setup complexity | Simple | Complex |
| Server requirements | SSH only | OpenVPN server |
| Encryption | SSH | SSL/TLS |
| Performance | Good | Excellent |
| Reliability | Good | Excellent |
| Configuration | Minimal | Extensive |

### sshuttle vs Chisel

| Feature | sshuttle | Chisel |
|---------|----------|--------|
| Protocol | SSH | HTTP/WebSocket |
| Setup | Simple | Moderate |
| Performance | Good | Better |
| Features | Routing only | Full tunneling |
| Encryption | SSH | TLS |

### sshuttle vs Proxychains

| Feature | sshuttle | Proxychains |
|---------|----------|-------------|
| Routing method | Transparent | LD_PRELOAD |
| Configuration | Minimal | Config file |
| DNS handling | Built-in | External |
| Performance | Better | Slightly worse |
| Compatibility | Linux only | Cross-platform |

## Resources

- [sshuttle GitHub](https://github.com/sshuttle/sshuttle)
- [sshuttle Documentation](https://sshuttle.readthedocs.io/)
- [Kali Linux sshuttle Page](https://www.kali.org/tools/sshuttle/)
- [MITRE ATT&CK Command and Control](https://attack.mitre.org/tactics/TA0011/)
- [SSH Protocol RFC 4253](https://tools.ietf.org/html/rfc4253)
- [iptables Documentation](https://www.netfilter.org/documentation/)
