---
tool_name: "pspy-binaries"
display_name: "Pspy Binaries"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "process_discovery"
install_command: "sudo apt install pspy"
version: "1.2.1"
github_repo: "https://github.com/DominicBreuker/pspy"
kali_tools_page: "https://www.kali.org/tools/pspy/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["process", "monitoring", "linux", "cron", "scheduled"]
related_tools: ["lsof", "ps", "top"]
---

# Pspy Binaries

## Chapter 1: Introduction & Overview

### What are Pspy Binaries?
Pspy Binaries are pre-compiled versions of Pspy for different architectures. They monitor Linux processes without root permissions.

### History & Background
- **Created:** 2018 by Dominic Breuker
- **Maintained:** Dominic Breuker
- **License:** GNU General Public License v3
- **Purpose:** Process monitoring

Pspy Binaries provide process monitoring.

### When to Use This Tool
- **Privilege escalation:** Find SUID binaries
- **Penetration testing:** Monitor processes
- **Security assessments:** Detect cron jobs
- **CTF challenges:** Linux enumeration

### Key Concepts
- **Process:** Running program
- **Binary:** Compiled executable
- **Architecture:** System type (32/64-bit)
- **Monitoring:** Watching changes

---

## Chapter 2: Installation & Setup

### System Requirements
- **OS:** Linux
- **Disk Space:** ~2 MB
- **Privileges:** User-level

### Installation Methods

#### Method 1: Binary Download
```bash
# Download from GitHub
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy32
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64

# Make executable
chmod +x pspy32 pspy64
```

### Verification
```bash
./pspy64 --version
```

---

## Chapter 3: Basic Usage

### First Use

#### Run Appropriate Binary
```bash
# Check architecture
uname -m

# Run 64-bit
./pspy64

# Run 32-bit
./pspy32
```

### Essential Commands

#### 64-bit System
```bash
./pspy64
```
**Explanation:** Monitors processes on 64-bit system.

#### 32-bit System
```bash
./pspy32
```
**Explanation:** Monitors processes on 32-bit system.

---

## Chapter 4: Intermediate Usage

### Advanced Usage

#### Background Monitoring
```bash
# Run in background
./pspy64 > /tmp/pspy.log 2>&1 &
```

#### Custom Interval
```bash
# Custom check interval
./pspy64 --interval 500
```

---

## Chapter 5: Advanced Usage

### Advanced Techniques

#### Combine with LinPEAS
```bash
# Monitor while running privilege escalation checks
./pspy64 &
./linpeas.sh
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: CTF Challenge
**Objective:** Find hidden processes

**Steps:**
```bash
# 1. Transfer binary
scp pspy64 user@target:/tmp/

# 2. Run monitoring
/tmp/pspy64

# 3. Wait for interesting processes
```

---

## Chapter 7: Detection & Defense

### Blue Team Response
1. **Monitor /tmp** - Detect binary execution
2. **Restrict file permissions** - Prevent execution
3. **Audit cron jobs** - Review scheduled tasks

---

## Chapter 8: Troubleshooting

### FAQ

**Q: Which binary should I use?**
A: Check architecture with `uname -m`. Use pspy64 for 64-bit, pspy32 for 32-bit.

**Q: How do I update pspy-binaries?**
A: Download latest releases from GitHub.

---

## Resources

### Related Tools
- **pspy:** Main tool
- **pspy64:** 64-bit binary
- **pspy32:** 32-bit binary

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
