---
tool_name: "ferret-sidejack"
display_name: "Ferret"
mitre_tactic: "collection"
mitre_tactic_id: "TA0009"
mitre_subcategory: "sniffing_and_cookie_harvesting"
install_command: "sudo apt install ferret-sidejack"
version: "3.0.1"
github_repo: "https://github.com/robertdavidgraham/ferret"
kali_tools_page: "https://www.kali.org/tools/ferret-sidejack/"
difficulty_level: "intermediate"
requires_root: true
gui_available: false
tags: ["sniffing", "cookie-harvesting", "sidejacking", "session-hijacking", "network-capture", "mitm"]
related_tools: ["hamster-sidejack", "wireshark", "tcpdump", "bettercap"]
---

# Ferret - Network Cookie & Credential Harvester

## Chapter 1: Introduction & Overview

### What is Ferret?

Ferret is a network monitoring tool that extracts interesting data from network traffic, specifically designed to harvest cookies, authentication tokens, and other session-related information from unencrypted network communications. It is the companion data-capture tool for the Hamster sidejacking framework — Ferret sniffs the wire, Hamster uses the captured data to hijack sessions.

### History & Background

- **Created:** Robert David Graham (Errata Security)
- **Version:** 3.0.1 (September 2024 build)
- **License:** Open source
- **Original Project:** Google Code (ferret.googlecode.com)
- **Language:** C/C++ with libpcap

Ferret was one of the pioneering session hijacking tools, demonstrating that unencrypted HTTP cookies on wired and wireless networks could be trivially captured and replayed. It works by passively sniffing network traffic and extracting cookies, HTTP headers, and authentication credentials from plaintext communications.

### When to Use This Tool

- **Penetration testing:** Demonstrate session hijacking risks on unencrypted networks
- **Red team engagements:** Capture credentials from internal network traffic
- **Wireless security assessments:** Show risks of open/unencrypted Wi-Fi
- **Network forensics:** Extract session data from pcap captures
- **CTF challenges:** Session hijacking and network sniffing challenges

### When NOT to Use This Tool

- Without network-level access authorization
- On production networks without written consent
- To intercept personal communications
- Against encrypted (HTTPS) traffic (Ferret captures plaintext only)

### Legal and Ethical Considerations

- **Network access authorization required** — sniffing traffic on unauthorized networks is illegal
- **Only works on unencrypted traffic** — HTTPS/TLS traffic is opaque to Ferret
- **Passive tool** — Ferret does not inject or modify traffic, it only reads
- **Data sensitivity** — captured cookies provide full account access
- **Scope limitations** — ensure the engagement includes network sniffing

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Sidejacking** | Hijacking a web session by stealing the session cookie |
| **Packet Capture** | Reading raw network packets via libpcap |
| **Cookie Extraction** | Parsing HTTP Set-Cookie headers from captured traffic |
| **Plaintext Protocols** | Only HTTP (not HTTPS) traffic yields usable cookies |
| **Offline Analysis** | Can process previously captured pcap files |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux recommended)
- **Root access:** Required for raw packet capture
- **Network interface:** Ethernet or Wi-Fi adapter in monitor mode (for wireless)
- **libpcap:** Packet capture library
- **Disk Space:** ~5 MB

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install ferret-sidejack
```

#### Method 2: Dependencies Only

```bash
# Install libpcap if not present
sudo apt install libpcap-dev libstdc++6
```

### Verify Installation

```bash
ferret-sidejack
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- libpcap version 1.10.6 (64-bit time_t, with TPACKET_V3)
options:
 -i <adapter>    Sniffs the wire(less) attached to that network adapter.
 -r <files>      Read files in off-line mode. Can use wildcards.
 -c <file>       Reads in more advanced parameters from a file.
```

### Network Interface Setup

```bash
# List available interfaces
ip link show

# For wireless - enable monitor mode
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up

# For ethernet - no special setup needed
# Promiscuous mode is set automatically by libpcap
```

---

## Chapter 3: Architecture & Operation

### Operation Modes

```
Mode 1: Live Capture
  Network Interface → libpcap → Packet Filter → HTTP Parser → Cookie Extractor → Output

Mode 2: Offline Analysis
  pcap File → Packet Parser → HTTP Parser → Cookie Extractor → Output

Mode 3: Feed to Hamster
  Network Interface → libpcap → Cookie Extractor → Hamster Proxy (port 1234)
```

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Network Sniffing | T1040 | Capture network traffic passively |
| Steal Web Session Cookie | T1539 | Extract session cookies from HTTP traffic |
| Man-in-the-Middle | T1557 | Position to intercept traffic |

### What Ferret Extracts

**HTTP Cookies:**
- Session identifiers (PHPSESSID, JSESSIONID, ASP.NET_SessionId, etc.)
- Authentication tokens
- Custom application cookies

**HTTP Headers:**
- Authorization headers (Basic, Bearer tokens)
- Custom authentication headers
- User-Agent strings (for session fingerprinting)

**Other Data:**
- Form POST data (usernames, passwords)
- Referer URLs
- Host headers

### Limitations

- **Plaintext only:** Cannot decrypt HTTPS/TLS traffic
- **Passive only:** Does not perform ARP spoofing or other MITM attacks
- **HTTP only:** Does not capture cookies from other protocols
- **No injection:** Does not modify or inject packets

---

## Chapter 4: Command Reference

### Basic Operation

#### Live Capture on Interface

```bash
sudo ferret-sidejack -i eth0
```

**Explanation:** Sniff all traffic on the eth0 interface and extract cookies/session data. Requires root privileges.

#### Live Capture on Wireless

```bash
sudo ferret-sidejack -i wlan0
```

**Explanation:** Sniff Wi-Fi traffic. Interface must be in monitor mode first.

#### Offline Pcap Analysis

```bash
ferret-sidejack -r capture.pcap
```

**Explanation:** Analyze a previously captured pcap file for cookies and session data. No root required for offline mode.

#### Multiple Pcap Files

```bash
ferret-sidejack -r *.pcap
```

**Explanation:** Process all pcap files in the current directory using wildcard expansion.

#### Custom Configuration File

```bash
ferret-sidejack -i eth0 -c ferret.conf
```

**Explanation:** Load advanced configuration from a file (filter rules, output format, etc.).

### Output and Logging

#### Direct Console Output

```bash
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_output.log
```

**Explanation:** Display captured data on console while simultaneously logging to file.

#### Combine with tcpdump for Reference Capture

```bash
# Terminal 1: Full packet capture for later analysis
sudo tcpdump -i eth0 -w full_capture.pcap

# Terminal 2: Ferret extraction
sudo ferret-sidejack -i eth0
```

**Explanation:** Run Ferret alongside a full packet capture for comprehensive evidence collection.

### Practical Usage Patterns

#### Capture on Internal Network Segment

```bash
# On the target network segment
sudo ferret-sidejack -i eth0 -c internal_network.conf 2>&1 | tee internal_capture.log
```

**Explanation:** Monitor an internal network segment for unencrypted session cookies.

#### Filter by MAC Address

```bash
# Use tcpdump filter before piping to Ferret
sudo tcpdump -i eth0 -U -w - 'src host 192.168.1.100' | ferret-sidejack -r -
```

**Explanation:** Filter traffic to a specific host before processing with Ferret.

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Wireless Session Harvesting

**Setup:**

```bash
# Step 1: Put wireless adapter in monitor mode
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up

# Step 2: Start Ferret
sudo ferret-sidejack -i wlan0 2>&1 | tee wireless_cookies.log

# Step 3: Feed captured cookies to Hamster
# In another terminal:
sudo hamster-sidejack
# Set browser proxy to http://127.0.0.1:1234
```

**Attack Flow:**
1. Wireless adapter captures all Wi-Fi traffic in range
2. Ferret extracts cookies from HTTP (non-HTTPS) sessions
3. Cookies are saved to ferret-capture.log (default location)
4. Hamster reads the log and uses cookies for session hijacking
5. Browser configured to proxy through Hamster accesses victim sessions

**Key Insight:** This attack only works against HTTP websites. HTTPS traffic is encrypted and yields no usable cookies. Many modern sites redirect HTTP to HTTPS, reducing the attack surface.

### Scenario 2: Internal Network Credential Capture

**Setup:**

```bash
# On a machine with access to the internal network
sudo ferret-sidejack -i eth0 2>&1 > /tmp/ferret_results.txt

# Analyze results
grep -i "cookie" /tmp/ferret_results.txt
grep -i "authorization" /tmp/ferret_results.txt
```

**Capture Targets:**
- Legacy HTTP admin panels
- Internal web applications without TLS
- Printer web interfaces
- IoT device management pages
- Development/staging servers

### Scenario 3: Pcap Analysis for Post-Incident Investigation

```bash
# Analyze traffic captured during an incident
ferret-sidejack -r incident_capture_001.pcap 2>&1 | tee incident_cookies.txt

# Search for specific session tokens
grep -i "session" incident_cookies.txt

# Extract all unique cookies
sort -u incident_cookies.txt | grep "Cookie:" > unique_cookies.txt
```

**Use Case:** After a breach, analyze network captures to determine what sessions were compromised.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Network Monitoring:**
- Promiscuous mode detection via ARP probe responses
- Unusual packet capture processes (pcap, ferret, tcpdump)
- Traffic patterns consistent with sniffing tools

**Host-Based Detection:**
- Process monitoring for ferret/pcap processes
- Network interface mode changes (monitor mode)
- Unusual network adapter configurations

**IDS/IPS Rules:**

```
# Suricata rule for detecting promiscuous mode
alert ip any any -> any any (msg:"Possible promiscuous mode detection"; \
  content:"|01 00 5e|"; sid:1000001; rev:1;)
```

### Evasion Techniques

1. **Passive Operation:** Ferret only reads traffic — no packets are sent, making it invisible to most detection
2. **Stealth Monitoring:** Use a passive network tap instead of a wireless adapter
3. **Encrypted Channel:** Monitor only HTTP traffic to avoid detection by TLS inspection
4. **Timing:** Capture during peak hours when network noise is high
5. **Low Profile:** Run on a device that would normally be on the network (e.g., a legitimate server)

### Blue Team Countermeasures

- **Enforce HTTPS everywhere:** Use HSTS headers to prevent HTTP fallback
- **Network segmentation:** Isolate sensitive traffic on separate VLANs
- **802.1X:** Require authentication for network access
- **Promiscuous mode detection:** Monitor for ARP anomalies
- **Encrypted DNS:** Prevent DNS manipulation that could redirect HTTP traffic

---

## Chapter 7: Integration with Other Tools

### Hamster-Sidejack Integration

Ferret and Hamster are designed to work together:

```bash
# Terminal 1: Ferret captures cookies
sudo ferret-sidejack -i wlan0

# Terminal 2: Hamster uses captured cookies
sudo hamster-sidejack
# Hamster listens on 127.0.0.1:1234
# Configure browser proxy: HTTP 127.0.0.1:1234
```

**Workflow:**
1. Ferret captures cookies from the network
2. Hamster reads Ferret's output file
3. Browser proxy routes HTTP through Hamster
4. Hamster replaces browser cookies with captured ones
5. Navigating to victim's sites shows their authenticated sessions

### Wireshark Analysis

```bash
# Capture with Wireshark, then analyze with Ferret
# In Wireshark: File > Export Specified Packets > Save as .pcap

# Then analyze with Ferret
ferret-sidejack -r wireshark_capture.pcap
```

### BetterCAP Integration

```bash
# BetterCAP performs ARP spoofing to position for sniffing
sudo bettercap -iface eth0

# In BetterCAP:
# set arp.spoof.targets 192.168.1.0/24
# arp.spoof on

# Ferret then captures the redirected traffic
sudo ferret-sidejack -i eth0
```

### tcpdump Correlation

```bash
# Full packet capture for forensic correlation
sudo tcpdump -i eth0 -w full_capture.pcap -c 100000

# Ferret extraction in parallel
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_extract.txt

# Correlate timestamps between full capture and Ferret output
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Permission denied on interface**

```bash
# Run with root privileges
sudo ferret-sidejack -i eth0

# Or grant capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/ferret-sidejack
```

**Issue: No cookies captured**

```bash
# Verify traffic is flowing
sudo tcpdump -i eth0 -c 10

# Check if traffic is encrypted (HTTPS)
sudo tcpdump -i eth0 -c 20 port 80

# If no HTTP traffic, most sites use HTTPS now
# Try targeting internal/legacy HTTP services
```

**Issue: Interface not found**

```bash
# List available interfaces
ip link show

# For monitor mode on wireless
sudo airmon-ng start wlan0
# Use the monitor interface (e.g., wlan0mon)
sudo ferret-sidejack -i wlan0mon
```

**Issue: Hamster not reading Ferret output**

```bash
# Check default output location
ls ~/hamster/cookies.txt
ls /tmp/ferret*.log

# Specify output explicitly
sudo ferret-sidejack -i eth0 -c custom_config.conf

# Verify Hamster is looking in the right directory
```

**Issue: Low capture rate**

```bash
# Increase capture buffer
sudo ferret-sidejack -i eth0 -c high_buffer.conf

# Use a dedicated capture interface
sudo tcpdump -i dedicated_nic -w capture.pcap &
sudo ferret-sidejack -r capture.pcap
```

### Debug Mode

```bash
# Verbose output for troubleshooting
sudo ferret-sidejack -i eth0 -v 2>&1 | tee debug.log

# Check libpcap version
ferret-sidejack 2>&1 | grep "libpcap"
```

### Performance Optimization

```bash
# For high-traffic networks, capture only HTTP
sudo tcpdump -i eth0 -w http_only.pcap 'tcp port 80'

# Analyze the filtered capture
ferret-sidejack -r http_only.pcap
```

---

## Resources

- **GitHub (Mirror):** https://github.com/robertdavidgraham/ferret
- **Errata Security:** http://www.erratasec.com/research.html
- **Kali Documentation:** https://www.kali.org/tools/ferret-sidejack/
- **Hamster-Sidejack:** https://www.kali.org/tools/hamster-sidejack/
- **Original Project:** http://ferret.googlecode.com (archived)
- **Wireshark:** https://www.wireshark.org
- **tcpdump:** https://www.tcpdump.org
- **Related - BetterCAP:** https://www.bettercap.org
