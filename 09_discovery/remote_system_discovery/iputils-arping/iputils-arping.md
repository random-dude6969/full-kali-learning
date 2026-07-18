---
tool_name: "iputils-arping"
display_name: "Iputils Arping"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "remote_system_discovery"
install_command: "sudo apt install iputils-arping"
version: "20210202"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/iputils-arping/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
tags: ["arp", "ping", "discovery", "host", "network"]
related_tools: ["arping", "fping", "ping"]
---

# Iputils Arping

## Chapter 1: Introduction & Overview

### What is Iputils Arping?
Iputils Arping is an ARP-level ping utility. It sends ARP requests to discover hosts on a local network.

### History & Background
- **Created:** 2002 by Alexey Kuznetsov
- **Maintained:** iputils project
- **License:** GNU General Public License v2
- **Purpose:** ARP-level ping

Iputils Arping sends ARP pings.

### When to Use This Tool
- **Network discovery:** Find local hosts
- **Penetration testing:** Discover network
- **Security assessments:** Map network
- **CTF challenges:** Host discovery

### Key Concepts
- **ARP:** Address Resolution Protocol
- **Layer 2:** Data link layer
- **Ping:** Network test
- **Host Discovery:** Finding devices

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~1 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install iputils-arping
```

### Verification
```bash
arping -h
```

---

## Chapter 3: Basic Usage

### First Use

#### Basic Arping
```bash
arping 192.168.1.1
```
**Output:** Sends ARP request.

### Essential Commands

#### Basic Ping
```bash
arping 192.168.1.1
```
**Explanation:** Sends ARP request.

#### Count
```bash
arping -c 5 192.168.1.1
```
**Explanation:** Sends 5 requests.

### Complete Flags Reference

#### Target Options
| Flag | Description | Example |
|------|-------------|---------|
| `-c` | Count | `arping -c 5 192.168.1.1` |
| `-I` | Interface | `arping -I eth0 192.168.1.1` |
| `-w` | Deadline | `arping -w 10 192.168.1.1` |
| `-W` | Wait time | `arping -W 1 192.168.1.1` |

#### Output Options
| Flag | Description | Example |
|------|-------------|---------|
| `-q` | Quiet | `arping -q 192.168.1.1` |
| `-v` | Verbose | `arping -v 192.168.1.1` |
| `-U` | Unsolicited | `arping -U 192.168.1.1` |

---

## Chapter 4: Intermediate Usage

### Advanced Discovery

#### Multiple Hosts
```bash
# Scan range
for i in $(seq 1 254); do
    arping -c 1 192.168.1.$i
done
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Network discovery
for i in $(seq 1 254); do
    if arping -c 1 -W 1 192.168.1.$i > /dev/null 2>&1; then
        echo "192.168.1.$i is up"
    fi
done
```

---

## Chapter 5: Advanced Usage

### Advanced Techniques

#### Unsolicited ARP
```bash
# Send unsolicited ARP
arping -U 192.168.1.100
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Network Discovery
**Objective:** Find live hosts

**Steps:**
```bash
# 1. Scan subnet
for i in $(seq 1 254); do
    arping -c 1 192.168.1.$i 2>/dev/null | grep "reply" | awk '{print $5}'
done | sort -u
```

---

## Chapter 7: Detection & Defense

### Blue Team Response
1. **Monitor ARP** - Detect scanning
2. **Use static ARP** - Prevent poisoning

---

## Chapter 8: Troubleshooting

### FAQ

**Q: Is iputils-arping legal to use?**
A: Yes, iputils-arping is a legitimate security testing tool. However, scanning without authorization is illegal.

**Q: How do I update iputils-arping?**
A: `sudo apt update && sudo apt upgrade iputils-arping`

---

## Resources

### Related Tools
- **arping:** Alternative arping
- **fping:** Ping multiple hosts
- **ping:** Standard ping

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
