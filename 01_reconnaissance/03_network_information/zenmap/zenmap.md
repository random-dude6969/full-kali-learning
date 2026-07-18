---
tool_name: "zenmap"
display_name: "Zenmap"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install zenmap"
version: "7.99"
github_repo: "https://github.com/nmap/zenmap"
kali_tools_page: "https://www.kali.org/tools/zenmap/"
difficulty_level: "beginner"
requires_root: false
gui_available: true
tags: ["gui", "nmap", "network-scanning", "visualization", "reconnaissance"]
related_tools: ["nmap", "masscan", "unicornscan"]
---

# Zenmap

## Chapter 1: Introduction and Overview

### What is Zenmap?

Zenmap is the official graphical user interface (GUI) for Nmap, the industry-standard network scanner. It provides a user-friendly way to run Nmap scans, visualize network topology, and compare scan results over time. Zenmap wraps all of Nmap's powerful functionality in an intuitive GUI, making network scanning accessible to users who may not be comfortable with command-line interfaces.

### History and Background

- **Original Author:** Gordon Lyon (Fyodor), Nmap creator
- **Language:** Python (using GTK+ for GUI)
- **License:** Nmap Public Source License (GPLv2-based)
- **Framework:** Built on top of Nmap
- **Cross-Platform:** Windows, Linux, macOS

Zenmap was first released alongside Nmap 4.0 in 2007. It was designed to make Nmap more accessible and to provide visual representations of network scans that are difficult to achieve with command-line output.

### When to Use This Tool

- **Visual network mapping:** When you need a graphical representation of network topology
- **Scan comparison:** Comparing scan results over time to detect changes
- **Learning Nmap:** Understanding Nmap options through the GUI
- **Report generation:** Creating visual reports for management or clients
- **Quick scans:** Running common scan profiles without memorizing flags
- **Multi-target management:** Managing scans against multiple targets simultaneously

### When NOT to Use This Tool

- On headless servers without display
- When scripting or automation is needed (use Nmap CLI)
- For high-volume scanning operations
- When resource-constrained (GUI uses more resources than CLI)

### Legal and Ethical Considerations

- Same legal requirements as Nmap — authorization required
- GUI output can be saved and shared — handle responsibly
- Visual network maps can reveal sensitive infrastructure details
- Follow the same ethical guidelines as command-line Nmap

### Key Concepts

- **Profile:** Pre-configured scan settings that can be saved and reused
- **Topology Map:** Visual representation of network hosts and connections
- **Scan Results:** Detailed output showing open ports, services, and OS info
- **Command Builder:** Interactive interface for constructing Nmap commands
- **Scan Comparison:** Side-by-side comparison of different scan results

---

## Chapter 2: Installation and Setup

### System Requirements

- **Operating System:** Linux, Windows, macOS
- **RAM:** 512 MB minimum (1 GB recommended)
- **Disk Space:** 100 MB for installation
- **Display:** X11/Wayland (Linux), any display (Windows/macOS)
- **Network:** Network interface for scanning
- **Privileges:** Root required for SYN scans, OS detection

### Installation Methods

#### Method 1: APT (Kali Linux - Recommended)

```bash
sudo apt update
sudo apt install zenmap
```

#### Method 2: From Source

```bash
git clone https://github.com/nmap/zenmap.git
cd zenmap
pip3 install -r requirements.txt
sudo python3 zenmap
```

#### Method 3: Nmap Bundle

```bash
# Zenmap is included with Nmap installer on Windows/macOS
# Download from: https://nmap.org/download.html
```

### Configuration

Zenmap stores its configuration in:

- **Linux:** `~/.zenmap/`
- **Windows:** `%APPDATA%\zenmap\`
- **macOS:** `~/Library/Application Support/zenmap/`

Key configuration files:
- `zenmap.conf` — Main configuration
- `scan_profiles.cfg` — Saved scan profiles
- `recent_scans.cfg` — Recent scan history

### Verification

```bash
zenmap --version
# Or launch from Applications menu
```

```bash
# Verify Nmap backend
nmap --version
# Output: Nmap 7.99
```

---

## Chapter 3: Basic Usage

### First Scan

1. Launch Zenmap from the application menu or terminal
2. Enter the target in the "Target" field (e.g., `scanme.nmap.org`)
3. Select a scan profile from the dropdown (e.g., "Intense scan")
4. Click "Scan"

**Output:** Results appear in the "Nmap Output" tab with a topology map in the "Topology" tab.

### Essential Commands via GUI

#### Quick Scan

1. Enter target: `192.168.1.1`
2. Profile: "Quick scan"
3. Click "Scan"

#### Intense Scan

1. Enter target: `192.168.1.1`
2. Profile: "Intense scan"
3. Click "Scan"

#### Network Scan

1. Enter target: `192.168.1.0/24`
2. Profile: "Intense scan, all TCP ports"
3. Click "Scan"

### Built-in Scan Profiles

| Profile | Command | Description |
|---------|---------|-------------|
| Intense scan | `nmap -T4 -A -v target` | Aggressive scan with OS detection |
| Intense scan + UDP | `nmap -sS -sU -T4 -A -v target` | TCP + UDP intensive scan |
| Intense scan, all TCP ports | `nmap -p 1-65535 -T4 -A -v target` | Full TCP port scan |
| Intense scan, no ping | `nmap -T4 -A -v -Pn target` | Skip host discovery |
| Ping scan | `nmap -sn target` | Host discovery only |
| Quick scan | `nmap -T4 -F target` | Fast scan of common ports |
| Quick scan plus | `nmap -T4 -O -sV target` | Quick + OS + version detection |
| Regular scan | `nmap target` | Default 1000 port scan |
| Slow comprehensive scan | `nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389,445 -PB -O --script all target` | Thorough scan |

### Command Builder

The Command Builder tab allows interactive Nmap command construction:

1. **Target Specification:** Enter hosts, IPs, or CIDR ranges
2. **Profile:** Select or create custom profiles
3. **Scan:** Click to execute the built command
4. **Filter:** Apply display filters to results

### GUI Tabs Overview

| Tab | Function |
|-----|----------|
| Nmap Output | Raw Nmap command output |
| Ports/Hosts | Table of discovered ports and services |
| Topology | Network topology visualization |
| Host Details | Detailed information per host |
| Scan Comparison | Compare multiple scans |

---

## Chapter 4: Intermediate Usage

### Custom Scan Profiles

#### Creating a Custom Profile

1. Open the "Profile" menu → "New Profile or Command"
2. Configure each tab:
   - **Target:** Set default target specification
   - **Scan:** Configure scan type, timing, scripts
   - **Ping:** Set host discovery options
   - **Scripting:** Configure NSE scripts
   - **Target Specification:** Set exclusions
   - **User Interface:** Configure display options
3. Save with a descriptive name

#### Example Custom Profiles

**Web Server Audit Profile:**
```bash
nmap -p 80,443,8080,8443 -sV -sC --script http-title,http-headers,ssl-cert -T4 target
```

**Windows Network Profile:**
```bash
nmap -p 135,139,445,3389 -sV -sC --script smb-enum-shares,smb-os-discovery -T4 target
```

**Stealth Profile:**
```bash
nmap -sS -T2 --max-retries 2 --host-timeout 300s target
```

### Topology Map Features

- **Zoom:** Mouse wheel or +/- buttons
- **Pan:** Click and drag
- **Select Host:** Click on node
- **Context Menu:** Right-click for host options
- **Color Coding:**
  - Green: Up host
  - Red: Down host
  - Blue: Scanned host
  - Gray: Unknown

### Scan Comparison

1. Run first scan, note results
2. Run second scan at later time
3. Go to "Scan Comparison" tab
4. Select both scans
5. View differences in hosts, ports, services

### Filter and Search

#### Display Filters

```
# Filter by port
port 80

# Filter by service
service http

# Filter by IP
host 192.168.1.1

# Filter by state
state open

# Combine filters
port 80 and service http
```

#### Search Results

- Use Ctrl+F to search within output
- Filter results by various criteria
- Export filtered results

---

## Chapter 5: Advanced Usage

### Script Configuration

#### Enabling NSE Scripts via GUI

1. Go to "Profile" → "Edit Profile"
2. Select "Scripting" tab
3. Check "Never use scripts" / "Default scripts" / "Custom scripts"
4. Select specific scripts from the list
5. Configure script arguments

#### Common Script Categories

| Category | Description | Example Scripts |
|----------|-------------|-----------------|
| auth | Authentication | ssh-brute, ftp-brute |
| broadcast | Network discovery | broadcast-arp |
| brute | Brute-force | http-brute, smb-brute |
| default | Standard scripts | default scripts |
| discovery | Service discovery | http-title, ssl-cert |
| dos | DoS testing | http-slowloris |
| exploit | Exploitation | smb-exploit |
| external | External services | whois-ip |
| fuzzer | Fuzzing | http-fuzz |
| intrusive | Invasive scripts | smb-brute |
| malware | Malware detection | ssl-heartbleed |
| safe | Safe scripts | safe scripts |
| version | Version detection | ssl-enum-ciphers |
| vuln | Vulnerability check | ssl-heartbleed |

### Custom Script Arguments

#### Through GUI

1. Edit Profile → Scripting
2. Select a script
3. Click "Script Args"
4. Enter arguments: `http-brute.path=/admin`

#### Common Script Arguments

| Script | Argument | Description |
|--------|----------|-------------|
| http-brute | `path=/admin` | Target path |
| ssh-brute | `userdb=users.txt` | User wordlist |
| smb-brute | `passdb=pass.txt` | Password wordlist |
| http-enum | `basepath=/` | Base path |
| ssl-cert | `sha256` | Show SHA256 |

### Integration with Other Tools

#### Export for Metasploit

```bash
# Export Nmap XML from Zenmap
# Import into Metasploit
msfconsole -x "db_import /path/to/scan.xml; hosts; services"
```

#### Python Scripting with Zenmap Data

```python
#!/usr/bin/env python3
# parse_zenmap.py - Parse Zenmap XML output

import xml.etree.ElementTree as ET

def parse_nmap_xml(xml_file):
    tree = ET.parse(xml_file)
    root = tree.getroot()

    for host in root.findall('.//host'):
        ip = host.find('address[@addrtype="ipv4"]').get('addr')
        state = host.find('status').get('state')

        if state == 'up':
            print(f"\nHost: {ip}")
            for port in host.findall('.//port'):
                portid = port.get('portid')
                protocol = port.get('protocol')
                state = port.find('state').get('state')
                service = port.find('service')
                service_name = service.get('name') if service else 'unknown'
                print(f"  {portid}/{protocol}: {state} - {service_name}")

# Usage
parse_nmap_xml('scan_results.xml')
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Audit for Small Business

**Objective:** Assess the external attack surface of a small business network.

**Steps:**

1. **Launch Zenmap** and enter the target network range
2. **Run Intense Scan** to discover all hosts and services
3. **Review Topology** to understand network layout
4. **Check Host Details** for each discovered system
5. **Compare scans** over time to detect changes
6. **Export results** for the security report

```bash
# Equivalent command-line operations
nmap -sS -sV -sC -O -p 1-65535 -T4 192.168.1.0/24 -oX network_audit.xml
```

### Scenario 2: CTF Challenge Reconnaissance

**Objective:** Quickly enumerate services on a CTF target.

1. **Quick Scan** to identify open ports
2. **Intense Scan** on discovered ports for service details
3. **Script Scan** for vulnerability detection
4. **Review output** for interesting findings

```bash
# CTF reconnaissance workflow
nmap -sV -sC -p- -T4 10.10.10.10 -oX ctf_scan.xml
```

### Scenario 3: Change Detection Monitoring

**Objective:** Monitor network for unauthorized changes.

1. **Baseline Scan:** Run comprehensive scan, save results
2. **Periodic Scans:** Repeat scan on schedule
3. **Compare:** Use Scan Comparison tab to identify changes
4. **Alert:** Investigate any new ports or services

```bash
# Baseline
nmap -sS -sV -p 1-1024 -T3 10.0.0.0/24 -oX baseline.xml

# Comparison scan (later)
nmap -sS -sV -p 1-1024 -T3 10.0.0.0/24 -oX current.xml

# Diff
ndiff baseline.xml current.xml
```

---

## Chapter 7: Detection and Defense

### Detection Signatures

Zenmap/Nmap activity can be detected through:

#### Network Indicators

- **SYN floods:** High volume of SYN packets from single source
- **Port sweep patterns:** Sequential port access
- **OS detection probes:** Specific TCP/IP stack behavior
- **NSE script traffic:** Characteristic request patterns

#### Log Indicators

```bash
# Detect Nmap-style scans in firewall logs
grep "SYN" /var/log/firewall.log | awk '{print $3}' | sort | uniq -c | sort -rn | head

# Check for port scanning patterns
grep -E "^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" /var/log/auth.log | \
    awk '{print $1}' | sort | uniq -c | sort -rn | head
```

### Mitigation Strategies

#### Firewall Rules

```bash
# Rate limit incoming SYN packets
iptables -A INPUT -p tcp --syn -m limit --limit 100/minute --limit-burst 200 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP

# Block Nmap user agents in web logs
grep -i "nmap" /var/log/nginx/access.log | awk '{print $1}' | \
    sort -u | xargs -I{} iptables -A INPUT -s {} -j DROP
```

#### Intrusion Detection

```yaml
# Suricata rule for Nmap detection
alert tcp any any -> $HOME_NET any (
    msg:"SCAN Nmap SYN scan detected";
    flags:S,12;
    threshold:type both,track by_src, count 100, seconds 10;
    sid:1000003;
    rev:1;
)
```

#### Network Architecture

- Deploy honeypots to detect scanning activity
- Use network segmentation to limit scan visibility
- Implement zero-trust architecture
- Monitor for unusual connection patterns

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Zenmap won't start"

**Cause:** Missing Python dependencies or display issues.

**Solution:**
```bash
# Install dependencies
pip3 install pygobject gtk

# Check display
echo $DISPLAY

# Try running from terminal for error messages
zenmap
```

#### Error: "Nmap not found"

**Cause:** Nmap not installed or not in PATH.

**Solution:**
```bash
# Install Nmap
sudo apt install nmap

# Verify Nmap location
which nmap

# Set path in Zenmap preferences
```

#### Error: "Permission denied"

**Cause:** Running SYN scans without root privileges.

**Solution:**
```bash
# Run Zenmap as root
sudo zenmap

# Or use Connect scan (-sT) which doesn't require root
```

#### Error: "Topology not displaying"

**Cause:** Insufficient data or rendering issues.

**Solution:**
- Ensure scan discovered multiple hosts
- Try zooming in/out
- Check graphics drivers
- Restart Zenmap

### Performance Optimization

- **Use profiles:** Save frequently-used scan configurations
- **Limit port ranges:** Avoid full port scans when possible
- **Reduce timing:** Use -T3 or lower for large networks
- **Close unused tabs:** Reduce memory usage
- **Export results:** Clear old scans from memory

### FAQ

**Q: Is Zenmap as powerful as Nmap CLI?**
A: Yes, Zenmap exposes all Nmap functionality through the GUI. However, CLI is better for scripting and automation.

**Q: Can I save and load scan profiles?**
A: Yes, use Profile → Save to create custom profiles, and Profile → Open to load them.

**Q: How do I compare two scans?**
A: Use the Scan Comparison tab to view differences between two saved scans.

**Q: Does Zenmap work on headless systems?**
A: No, Zenmap requires a graphical display. Use Nmap CLI on headless systems.

**Q: Can I export topology maps?**
A: Yes, right-click the topology and select "Save as" to export as image.

---

## Resources

### Official Documentation

- **Zenmap Official Site:** https://nmap.org/zenmap/
- **Nmap Documentation:** https://nmap.org/docs.html
- **Kali Linux Tools Page:** https://www.kali.org/tools/zenmap/

### Books

- *Nmap Network Scanning* by Gordon Lyon (Fyodor)
- *Penetration Testing* by Georgia Weidman
- *Network Security Assessment* by Chris McNab

### Video Tutorials

- **Zenmap Tutorial** by Nmap Official
- **Nmap GUI Walkthrough** by HackerSploit
- **Network Scanning with Zenmap** by NetworkChuck

### Practice Labs

- **HackTheBox:** Practice scanning target machines
- **TryHackMe:** Nmap learning rooms
- **ScanMe.Nmap.org:** Official Nmap test target
- **VulnHub:** Downloadable vulnerable machines
