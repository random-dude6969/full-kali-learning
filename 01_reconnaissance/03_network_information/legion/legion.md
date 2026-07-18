---
tool_name: "legion"
display_name: "Legion"
mitre_tactic: "reconnaissance"
mitre_tactic_id: "TA0043"
mitre_subcategory: "network_information"
install_command: "sudo apt install legion"
version: "0.3.5"
github_repo: "https://github.com/GoVanguard/legion"
kali_tools_page: "https://www.kali.org/tools/legion/"
difficulty_level: "beginner"
requires_root: false
gui_available: true
tags: ["recon", "scanner", "network", "automated"]
related_tools: ["nmap", "autorecon", "legion"]
---

# Legion

## Chapter 1: Introduction & Overview

### What is Legion?
Legion is an open-source, semi-automated network reconnaissance tool with a graphical interface. It provides automated service discovery, version detection, and vulnerability identification across network targets. It wraps multiple tools including Nmap, Hydra, and Nikto into an integrated GUI.

### History & Background
- **Created:** 2018 by GoVanguard
- **Maintained:** GoVanguard team
- **License:** AGPL-3.0
- **Language:** Python 3 + PyQt5
- **Purpose:** Semi-automated network reconnaissance with GUI
- **Architecture:** Modular plugin system with integrated tools
- **Features:** Service detection, vulnerability scanning, brute-force

### When to Use This Tool
- **Network discovery:** Map hosts and services on a network
- **Penetration testing:** Automated service enumeration
- **Quick scans:** Fast assessment of target networks
- **Non-technical users:** GUI-based alternative to command-line tools
- **CTF challenges:** Discover services and vulnerabilities

### When NOT to Use This Tool
- Without written authorization
- On production networks without planning
- For aggressive scanning that may disrupt services
- For any illegal activity

### Legal and Ethical Considerations
- Always obtain written permission before scanning
- Use appropriate scan timing for the environment
- Document all scanning activities
- Stay within the defined scope of engagement
- Consider the impact on target infrastructure
- Avoid denial-of-service conditions

### Key Concepts
- **Service enumeration:** Identifying running services and versions
- **Automated scanning:** Tool that chains multiple reconnaissance steps
- **GUI:** Graphical user interface for easier interaction
- **Nmap:** Network exploration and security auditing utility
- **NSE:** Nmap Scripting Engine for automated tasks
- **Brute-force:** Automated password guessing attack

---

## Chapter 2: Installation & Setup

### System Requirements
- **Python:** 3.6+
- **RAM:** 2 GB recommended
- **Disk Space:** 500 MB
- **Display:** X11/GUI environment
- **Privileges:** Root for some scans
- **OS:** Linux (Kali recommended)

### Installation

#### Method 1: APT
```bash
sudo apt update
sudo apt install legion
```
**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  legion
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 2.3 MB of archives.
After this operation, 8.9 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 legion all 0.3.5-1kali1 [2.3 MB]
Fetched 2.3 MB in 0.8s (2.9 MB/s)
Selecting previously unselected package legion.
(Reading database ... 1234567 files and directories currently installed.)
Preparing to unpack .../legion_0.3.5-1kali1_all.deb ...
Unpacking legion (0.3.5-1kali1) ...
Setting up legion (0.3.5-1kali1) ...
Processing triggers for man-db (2.10.2-1) ...
```

#### Method 2: From Source
```bash
git clone https://github.com/GoVanguard/legion.git
cd legion
pip3 install -r requirements.txt
```
**Expected output:**
```
Cloning into 'legion'...
remote: Enumerating objects: 4567, done.
remote: Counting objects: 100% (456/456), done.
remote: Compressing objects: 100% (189/189), done.
remote: Total 4567 (delta 345), reused 400 (delta 300)
Receiving objects: 100% (4567/4567), 3.45 MiB | 4.12 MiB/s, done.
Resolving deltas: 100% (345/345), done.
Collecting PyQt5>=5.15.0
  Downloading PyQt5-5.15.7-cp37-abi3-manylinux1_0_x86_64.whl (8.4 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 8.4/8.4 MB 5.67 MB/s eta 0:00:00
Installing collected packages: PyQt5, PyQt5-sip, PyQt5-qt5
Successfully installed PyQt5-5.15.7 PyQt5-sip-12.11.0 PyQt5-qt5-5.15.7
```

### Launch
```bash
sudo legion
```
**Expected output:**
```
Legion v0.3.5 - Semi-automated Network Reconnaissance
[*] Initializing GUI...
[*] Loading Nmap integration...
[*] Loading service plugins...
[*] Legion ready on display :0
```

### GUI Interface Overview
```
+----------------------------------------------------------+
| Legion v0.3.5                              [Settings] [?] |
+----------------------------------------------------------+
| [New Scan] [Open Project] [Save] [Export]                 |
+----------------------------------------------------------+
| Targets        | Services       | Vulnerabilities         |
|----------------|----------------|-------------------------|
| 192.168.1.1    | HTTP (80)      | CVE-2021-XXXX           |
| 192.168.1.2    | SSH (22)       | None found              |
| 192.168.1.3    | FTP (21)       | CVE-2020-YYYY           |
+----------------------------------------------------------+
| Tool Output                                               |
| [*] Nmap scan started...                                  |
| [*] Host discovery completed                              |
+----------------------------------------------------------+
| Status: Scanning... | Hosts: 3 | Services: 8             |
+----------------------------------------------------------+
```

---

## Chapter 3: Basic Usage

### Starting a Scan
1. Launch Legion: `sudo legion`
2. Click "New Scan"
3. Enter target IP or range
4. Select scan profile
5. Click "Start"

### Essential Features

#### Host Discovery
- Automatically discovers live hosts
- Displays open ports and services
- Identifies operating systems

#### Service Detection
- Identifies service versions
- Matches against vulnerability databases
- Detects web servers and applications

#### Automated Brute Force
- Tests default credentials
- Configurable username/password lists
- Supports multiple protocols

### GUI Overview
- **Target Panel:** Shows discovered hosts
- **Service Panel:** Lists detected services
- **Vulnerability Panel:** Shows potential vulnerabilities
- **Tool Output:** Raw tool output
- **Status Bar:** Scan progress and statistics

### Command-Line Alternatives
```bash
# Start Legion with specific options
sudo legion --target 192.168.1.0/24 --profile full

# Run scan in background
sudo legion --target target.com --profile quick &
```

---

## Chapter 4: Intermediate Usage

### Custom Scan Profiles
1. Go to Settings → Scan Profiles
2. Create new profile
3. Configure Nmap options
4. Set service-specific tools

### Scripting Integration
```python
# Legion uses Python for extensions
# Custom modules can be added to /usr/share/legion/modules/

# Example custom module
class CustomScanner:
    def __init__(self):
        self.name = "Custom Scanner"
    
    def scan(self, target):
        # Custom scanning logic
        return results
```

### Output Management
- Export scan results to XML/HTML
- Generate reports
- Save and load scan projects

### Advanced Configuration
```bash
# Configure Legion via config files
sudo nano /etc/legion/config.conf

# Set default scan options
# Adjust timeouts and retry counts
# Configure notification settings
```

---

## Chapter 5: Advanced Usage

### Custom NSE Scripts
```bash
# Legion uses Nmap NSE scripts
# Configure in scan profile settings

# Example NSE script usage
nmap --script=default,vuln -p- target.com

# Custom NSE script location
ls /usr/share/nmap/scripts/
```

### Integration with Other Tools
- **Nmap:** Port scanning and service detection
- **Hydra:** Password brute-forcing
- **Nikto:** Web vulnerability scanning
- **Gobuster:** Directory brute-forcing
- **SSLScan:** SSL/TLS configuration analysis

### Docker Deployment
```bash
# Run Legion in Docker with GUI support
docker run -it --rm \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  legion:latest
```

### Automated Reporting
```bash
# Generate HTML report from scan results
python3 -c "
import xml.etree.ElementTree as ET
tree = ET.parse('scan_results.xml')
root = tree.getroot()
print('Scan Report')
print('===========')
for host in root.findall('.//host'):
    addr = host.find('address').get('addr')
    print(f'Host: {addr}')
"
```

### Scan Profile Comparison
| Profile | Speed | Coverage | Use Case |
|---------|-------|----------|----------|
| Quick Scan | Fast | Basic ports | Initial discovery |
| Full Scan | Slow | All ports + scripts | Comprehensive assessment |
| Full and Fast | Medium | Optimized NVTs | Balanced approach |
| Discovery | Fast | Host enumeration | Network mapping |
| Vulnerability | Slow | Security checks | Vulnerability assessment |

### Nmap Command Reference
```bash
# Quick scan
nmap -F target.com

# Full port scan
nmap -p- target.com

# Service version detection
nmap -sV target.com

# OS detection
nmap -O target.com

# Script scan
nmap --script=default target.com

# Aggressive scan
nmap -A target.com
```

### Legion Module System
```bash
# List available modules
ls /usr/share/legion/modules/

# Module categories:
# - discovery/    : Host and service discovery
# - enumeration/  : Service enumeration
# - exploitation/ : Vulnerability testing
# - reporting/    : Report generation

# Custom module example
cat > /usr/share/legion/modules/custom_scanner.py << 'EOF'
class CustomScanner:
    def __init__(self):
        self.name = "Custom Scanner"
        self.version = "1.0"
    
    def scan(self, target, ports):
        results = []
        for port in ports:
            # Custom scanning logic
            results.append({"port": port, "status": "open"})
        return results
EOF
```

### Integration with External Tools
```bash
# Hydra brute-force integration
hydra -L users.txt -P passwords.txt ssh://target.com

# Nikto web scanner
nikto -h https://target.com

# Gobuster directory brute-force
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Assessment
```bash
# Step 1: Launch Legion
sudo legion

# Step 2: Create new scan → Enter target range
# Step 3: Select "Full Scan" profile
# Step 4: Review results in Service panel
# Step 5: Export report for documentation

# Command-line equivalent:
nmap -sV -sC -p- -oX scan_results.xml 192.168.1.0/24
```

### Scenario 2: Quick Service Check
```bash
# Step 1: New scan → Single host
# Step 2: Select "Quick Scan" profile
# Step 3: Review open ports and services

# Quick scan command:
nmap -F -sV target.com
```

### Scenario 3: Penetration Test
```bash
# Step 1: Full network scan
sudo legion --target 10.0.0.0/24 --profile full

# Step 2: Review service versions
# Step 3: Run automated brute force on found services
# Step 4: Document findings

# Post-scan analysis:
grep -r "open" scan_results/ | sort -u
```

### Scenario 4: CTF Challenge
```bash
# CTF: Find hidden services
sudo legion --target ctf-target.com --profile full

# Alternative command-line approach:
nmap -sV -sC -p- --script=default,vuln ctf-target.com
```

---

## Chapter 7: Detection & Defense

### Detection
- Nmap scan signatures detected by IDS/IPS
- Multiple service probe attempts
- Brute-force login attempts
- Sequential port scan patterns
- OS fingerprinting attempts

### IDS/IPS Signatures
```
# Snort rule for Nmap scan detection
alert tcp any any -> any any (msg:"Nmap Scan Detected"; \
  flags:S; threshold:type both, track by_src, count 100, seconds 60; \
  classtype:reconnaissance; sid:1000001; rev:1;)

# Suricata rule for service enumeration
alert tcp any any -> any any (msg:"Service Enumeration"; \
  content:"GET"; offset:0; depth:4; \
  threshold:type both, track by_src, count 50, seconds 60; \
  classtype:reconnaissance; sid:1000002; rev:1;)
```

### Mitigation
- Deploy IDS/IPS to detect scanning
- Implement account lockout policies
- Monitor for sequential port probes
- Use network segmentation to limit access
- Deploy honeypots to detect reconnaissance
- Set up alerts for unusual scan patterns

### Blue Team Response
1. Monitor for rapid port scan patterns
2. Log and alert on service enumeration attempts
3. Implement progressive delays for repeated connections
4. Deploy network segmentation to limit lateral movement
5. Use threat intelligence feeds to detect known scanner IPs
6. Set up canary tokens to detect reconnaissance activity

---

## Chapter 8: Troubleshooting

### Common Errors

#### "Display not found"
**Solution:** Ensure X11 forwarding or local GUI session.
```bash
# For SSH with X11 forwarding
ssh -X user@host

# Set display variable
export DISPLAY=:0
```

#### "Permission denied"
**Solution:** Run with `sudo` for full functionality.
```bash
sudo legion
```

#### "No hosts found"
**Solution:** Check network connectivity and firewall rules.
```bash
# Verify network connectivity
ping -c 3 target.com

# Check if target is reachable
nmap -sn target.com
```

#### "Nmap not found"
**Solution:** Install Nmap:
```bash
sudo apt install nmap
```

### Performance Tips
- Use targeted scan profiles for faster results
- Limit port ranges for quicker scans
- Save scan projects for later analysis
- Use multiple scan profiles for comprehensive coverage

### FAQ
**Q: What's the difference between Legion and Nmap?**
A: Legion provides a GUI and automated tool chaining, while Nmap is command-line only.

**Q: Can Legion run without root?**
A: Some features require root. Use `sudo` for full functionality.

**Q: How do I save scan results?**
A: Use File → Save Project to save all scan data.

**Q: How do I update Legion?**
A: Run `sudo apt update && sudo apt upgrade legion` or `git pull && pip3 install -r requirements.txt`.

---

## Resources

### Official Documentation
- [Legion GitHub](https://github.com/GoVanguard/legion)
- [Nmap Documentation](https://nmap.org/docs.html)
- [Legion Wiki](https://github.com/GoVanguard/legion/wiki)

### Related Tools
- **Nmap** — Network scanner
- **AutoRecon** — Automated network reconnaissance
- **Masscan** — Fast port scanner
- **OpenVAS** — Vulnerability scanner

### Practice
- Use with authorized penetration testing engagements
- Practice on intentionally vulnerable applications
- Join security communities for tips and techniques
