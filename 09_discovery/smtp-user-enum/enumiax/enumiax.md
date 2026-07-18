---
tool_name: "enumiax"
display_name: "EnumIAX"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "voip"
install_command: "sudo apt install enumiax"
version: "0.0.3"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/enumiax/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["voip", "iax", "asterisk", "enumeration", "extensions"]
related_tools: ["sipvicious", "sipp", "ohrwurm"]
---

# EnumIAX

## Chapter 1: Introduction & Overview

### What is EnumIAX?
EnumIAX is a tool for enumerating IAX (Inter-Asterisk eXchange) extensions on Asterisk servers. It discovers valid extensions through brute-force.

### History & Background
- **Created:** 2007 by EnumIAX
- **Maintained:** EnumIAX
- **License:** GNU General Public License v2
- **Purpose:** IAX extension enumeration

EnumIAX discovers IAX extensions.

### When to Use This Tool
- **VoIP auditing:** Find IAX extensions
- **Penetration testing:** Enumerate Asterisk
- **Security assessments:** Test VoIP security
- **CTF challenges:** VoIP challenges

### Key Concepts
- **IAX:** Inter-Asterisk eXchange protocol
- **Asterisk:** PBX system
- **Extension:** User number
- **Brute-force:** Try all combinations

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install enumiax
```

### Verification
```bash
enumiax --help
```

---

## Chapter 3: Basic Usage

### First Use

#### Enumerate Extensions
```bash
enumiax -h 192.168.1.1
```
**Output:** Enumerates IAX extensions.

### Essential Commands

#### Basic Enumeration
```bash
enumiax -h 192.168.1.1
```
**Explanation:** Enumerates IAX extensions.

---

## Chapter 4: Intermediate Usage

### Advanced Enumeration

#### Custom Range
```bash
# Enumerate specific range
enumiax -r 100-200 -h 192.168.1.1
```

---

## Chapter 5: Advanced Usage

### Advanced Techniques

#### Multiple Targets
```bash
# Enumerate multiple targets
for target in 192.168.1.{1..10}; do
    enumiax -h $target
done
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: VoIP Audit
**Objective:** Find IAX extensions

**Steps:**
```bash
# 1. Enumerate extensions
enumiax -h 192.168.1.1

# 2. Note found extensions
# 3. Brute force passwords
```

---

## Chapter 7: Detection & Defense

### Blue Team Response
1. **Change default extensions** - Avoid sequential numbers
2. **Implement authentication** - Require passwords
3. **Monitor IAX traffic** - Detect enumeration

---

## Chapter 8: Troubleshooting

### FAQ

**Q: Is enumiax legal to use?**
A: Yes, enumiax is a legitimate security testing tool. However, enumerating without authorization is illegal.

**Q: How do I update enumiax?**
A: `sudo apt update && sudo apt upgrade enumiax`

---

## Resources

### Related Tools
- **sipvicious:** VoIP toolkit
- **sipp:** SIP protocol tester

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
