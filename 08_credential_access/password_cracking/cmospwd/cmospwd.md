---
tool_name: "cmospwd"
display_name: "CMOSPwd"
mitre_tactic: "credential_access"
mitre_tactic_id: "TA0006"
mitre_subcategory: "password_cracking"
install_command: "sudo apt install cmospwd"
version: "7.0"
github_repo: "https://github.com/edge-security/cmospwd"
kali_tools_page: "https://www.kali.org/tools/cmospwd/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: false
tags: ["password", "cracking", "bios", "cmos", "firmware"]
related_tools: ["john", "hashcat", "ophcrack"]
---

# CMOSPwd

## Chapter 1: Introduction & Overview

### What is CMOSPwd?
CMOSPwd is a tool designed to decrypt or recover passwords stored in BIOS/CMOS. It supports a wide range of BIOS manufacturers including Award, Phoenix, AMI, and others. The tool can recover lost or forgotten BIOS passwords by analyzing the CMOS memory dump.

### History & Background
- **Created:** 2005 by edge-security
- **Maintained:** edge-security
- **License:** GNU General Public License v2
- **Purpose:** BIOS/CMOS password recovery

CMOSPwd is essential for recovering lost BIOS passwords during forensic investigations or when locked out of systems.

### When to Use This Tool
- **Forensics:** Recover BIOS passwords from systems
- **Incident response:** Gain access to locked systems
- **Penetration testing:** Test BIOS security
- **IT support:** Recover forgotten BIOS passwords

### Key Concepts
- **BIOS:** Basic Input/Output System
- **CMOS:** Complementary Metal-Oxide Semiconductor
- **EEPROM:** Electrically Erasable Programmable Read-Only Memory
- **Password Recovery:** Decrypting stored passwords

---

## Chapter 2: Installation & Setup

### System Requirements
- **Disk Space:** ~5 MB
- **Privileges:** User-level
- **Access:** CMOS dump file (for offline recovery)

### Installation Methods

#### Method 1: APT (Recommended)
```bash
sudo apt update
sudo apt install cmospwd
```

#### Method 2: From Source
```bash
git clone https://github.com/edge-security/cmospwd.git
cd cmospwd
make
```

### Verification
```bash
cmospwd --help
```

---

## Chapter 3: Basic Usage

### First Recovery

#### Recover BIOS Password
```bash
cmospwd /user
```
**Output:** Attempts to recover BIOS password from system.

### Essential Commands

#### Interactive Mode
```bash
cmospwd
```
**Explanation:** Opens interactive mode for BIOS password recovery.

#### Command Line Recovery
```bash
cmospwd /user
```
**Explanation:** Recovers BIOS user password.

#### All Passwords
```bash
cmospwd /all
```
**Explanation:** Recovers all BIOS passwords.

### Complete Command Reference

#### Recovery Options
| Command | Description | Example |
|---------|-------------|---------|
| `/user` | User password | `cmospwd /user` |
| `/admin` | Admin password | `cmospwd /admin` |
| `/all` | All passwords | `cmospwd /all` |
| `/debug` | Debug mode | `cmospwd /debug` |

#### BIOS Manufacturer Support
| Manufacturer | Command | Example |
|--------------|---------|---------|
| Award | `/award` | `cmospwd /award` |
| Phoenix | `/phoenix` | `cmospwd /phoenix` |
| AMI | `/ami` | `cmospwd /ami` |
| Compaq | `/compaq` | `cmospwd /compaq` |
| Dell | `/dell` | `cmospwd /dell` |
| Hewlett-Packard | `/hp` | `cmospwd /hp` |
| IBM | `/ibm` | `cmospwd /ibm` |

#### Output Options
| Command | Description | Example |
|---------|-------------|---------|
| `/o` | Output file | `cmospwd /user /o results.txt` |
| `/v` | Verbose | `cmospwd /user /v` |

---

## Chapter 4: Intermediate Usage

### Advanced Recovery Techniques

#### Manufacturer-Specific Recovery
```bash
# Award BIOS
cmospwd /award

# Phoenix BIOS
cmospwd /phoenix

# AMI BIOS
cmospwd /ami
```

#### CMOS Dump Recovery
```bash
# Recover from CMOS dump file
cmospwd /d dump_file.bin
```

### Scripting & Automation

#### Bash Scripting
```bash
#!/bin/bash
# Automated BIOS password recovery
echo "[*] Attempting BIOS password recovery..."
cmospwd /all
```

### Combining with Other Tools

#### With Hardware Tools
```bash
# Use with hardware tools for CMOS extraction
# Then recover password
cmospwd /d extracted_cmos.bin
```

### Advanced Configurations

#### Custom CMOS Dump
```bash
# Recover from custom CMOS dump
cmospwd /d custom_dump.bin /v
```

---

## Chapter 5: Advanced Usage

### Advanced Recovery Techniques

#### Multiple BIOS Types
```bash
# Try all BIOS types
cmospwd /all

# Debug mode
cmospwd /all /debug
```

### Integration in Pentest Workflow

#### Phase 1: CMOS Extraction
```bash
# Extract CMOS (hardware method or from system)
# Save as dump file
```

#### Phase 2: Password Recovery
```bash
# Recover password
cmospwd /d dump_file.bin
```

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Forensic Investigation
**Objective:** Recover BIOS password for evidence access

**Steps:**
```bash
# 1. Extract CMOS from system
# 2. Recover password
cmospwd /all
# 3. Use recovered password to access BIOS
```

### Scenario 2: IT Support
**Objective:** Recover forgotten BIOS password

**Steps:**
```bash
# 1. Access system
# 2. Run recovery tool
cmospwd /user
# 3. Reset BIOS with recovered password
```

---

## Chapter 7: Detection & Defense

### How Defenders Detect CMOSPwd

#### System Indicators
```
# BIOS access attempts
# CMOS dump files
```

### Mitigation Strategies

#### Strong BIOS Security
```bash
# Set strong BIOS passwords
# Enable TPM
# Enable Secure Boot
```

### Blue Team Response
1. **Set strong BIOS passwords** - Complex passwords
2. **Enable hardware security** - TPM, Secure Boot
3. **Monitor BIOS access** - Detect unauthorized attempts
4. **Physical security** - Prevent hardware access

---

## Chapter 8: Troubleshooting

### Common Errors

#### Error: "No BIOS detected"
**Cause:** BIOS not supported or not accessible
**Solution:**
```bash
# Try manufacturer-specific command
cmospwd /award
cmospwd /phoenix
cmospwd /ami
```

#### Error: "Access denied"
**Cause:** Insufficient privileges
**Solution:**
```bash
# Run with appropriate privileges
sudo cmospwd /user
```

### FAQ

**Q: Is cmospwd legal to use?**
A: Yes, cmospwd is a legitimate security testing tool. However, using it without authorization is illegal.

**Q: What BIOS types are supported?**
A: Award, Phoenix, AMI, Compaq, Dell, HP, IBM, and more.

**Q: Can cmospwd recover all BIOS passwords?**
A: Most modern BIOS types are supported, but some newer systems may not be compatible.

---

## Resources

### Official Documentation
- [CMOSPwd GitHub](https://github.com/edge-security/cmospwd)
- [CMOSPwd Documentation](https://github.com/edge-security/cmospwd/wiki)

### Related Tools
- **john:** John the Ripper
- **hashcat:** Advanced password recovery
- **ophcrack:** Windows password cracker

### Practice Resources
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe](https://tryhackme.com/)
