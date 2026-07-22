# chkrootkit — Check for Rootkits

## Table of Contents

1. [Overview](#overview)
2. [Understanding Rootkits](#understanding-rootkits)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Available Tests](#available-tests)
7. [Understanding the Output](#understanding-the-output)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

**chkrootkit** (Check Rootkit) is a security scanner that searches for signs of rootkit infection on a Linux system. It checks for over 70 different rootkits and provides indicators of compromise.

Rootkits are malicious software designed to provide continued privileged access to a computer while hiding their presence from users and administrators. chkrootkit detects rootkits by:

- Checking system binaries for modifications
- Looking for known rootkit files and directories
- Examining network connections for suspicious activity
- Checking for modified system utilities
- Detecting hidden processes and files

### Important Note

No automated tool can guarantee a system is uncompromised. chkrootkit is a valuable first step, but human judgment and additional investigation are always needed.

---

## Understanding Rootkits

### What Is a Rootkit?

A **rootkit** is a collection of software tools that enables an unauthorized user to gain control of a computer system without being detected. Rootkits can:

- Hide processes from task managers
- Hide files from directory listings
- Intercept system calls
- Modify system utilities (ps, ls, netstat, etc.)
- Provide backdoor access
- Steal credentials

### Types of Rootkits

| Type | Location | Detection Method |
|------|----------|-----------------|
| User-mode | Userspace | Check system binaries |
| Kernel-mode | Kernel space | Check kernel modules |
| Bootkit | Boot sector | Check boot records |
| Hypervisor | Below OS | Check hardware |
| Firmware | Device firmware | Check device firmware |

### Known Rootkits Detected by chkrootkit

chkrootkit can detect signs of over 70 rootkits including:

- Adore
- Bandit
- Backdoor ( variants)
- Chinese
- Enye LKM
- Flea
- Knark
- LKM (Loadable Kernel Module)
- MonKit
- N60
- OpticKit
- RKF
- Sebek
- SucKit
- TuxKit
- ZK

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install chkrootkit
```

### Verify Installation

```bash
chkrootkit -V
```

### Configuration

```bash
# Configuration file
cat /etc/chkrootkit/chkrootkit.conf

# Daily scan configuration
cat /etc/chkrootkit/chkrootkit.conf
```

---

## Command Syntax

```bash
chkrootkit [options] [tests]
```

### Basic Usage

```bash
# Run all tests
sudo chkrootkit

# Run with specific options
sudo chkrootkit -x -q

# Run specific test
sudo chkrootkit suspscan
```

---

## Options Reference

| Option | Description |
|--------|-------------|
| `-h` | Show help |
| `-V` | Show version |
| `-l` | Show available tests |
| `-d` | Debug mode |
| `-q` | Quiet mode (only suspicious output) |
| `-x` | Expert mode (more thorough) |
| `-e 'FILE1 FILE2'` | Exclude files/dirs from results |
| `-s REGEXP` | Filter sniffer test results |
| `-r DIR` | Use DIR as root directory |
| `-p DIR1:DIR2:DIRN` | Path for external commands |
| `-n` | Skip NFS mount points |
| `-T FSTYPE` | Skip mount points of specified type |

### Expert Mode (`-x`)

Expert mode enables more thorough testing:

```bash
# Run in expert mode
sudo chkrootkit -x

# Expert mode with quiet output
sudo chkrootkit -x -q
```

### Excluding False Positives

```bash
# Exclude known safe files
sudo chkrootkit -e '/usr/bin/vmstat /usr/bin/lsof'

# Use README.FALSE-POSITIVES for guidance
cat /usr/share/doc/chkrootkit/README.FALSE-POSITIVES
```

---

## Available Tests

### Listing Tests

```bash
# Show all available tests
chkrootkit -l
```

### Key Tests

| Test | Description |
|------|-------------|
| `suspscan` | Scans for suspicious files |
| `bindshell` | Checks for backdoor shells |
| `chfn` | Checks chfn binary |
| `chsh` | Checks chsh binary |
| `cron` | Checks cron daemon |
| `date` | Checks date binary |
| `du` | Checks du binary |
| `echo` | Checks echo binary |
| `env` | Checks env binary |
| `find` | Checks find binary |
| `finger` | Checks finger binary |
| `head` | Checks head binary |
| `lastlog` | Checks lastlog for anomalies |
| `lsof` | Checks lsof binary |
| `mail` | Checks mail binary |
| `netscan` | Scans for suspicious network connections |
| `passwd` | Checks passwd binary |
| `pidof` | Checks pidof binary |
| `ps` | Checks ps binary |
| `pstree` | Checks pstree binary |
| `shellshock` | Checks for Shellshock vulnerability |
| `sshd` | Checks sshd configuration |
| `strings` | Checks strings binary |
| `top` | Checks top binary |
| `touch` | Checks touch binary |
| `ulimit` | Checks ulimit binary |
| `utmp` | Checks utmp for anomalies |
| `w` | Checks w binary |
| `write` | Checks write binary |

### Network Tests

| Test | Description |
|------|-------------|
| `netscan` | Scans for suspicious connections |
| `sniffer` | Checks for network sniffers |

### Log Tests

| Test | Description |
|------|-------------|
| `chklastlog` | Checks lastlog for deleted entries |
| `chkwtmp` | Checks wtmp for deleted entries |

---

## Understanding the Output

### Clean Output

```
ROOTDIR is: /
Checking `amd64-microcode'... not found
Checking `at'... not found
Checking `bindshell'... not infected
Checking `chfn'... not infected
```

### Suspicious Output

```
ROOTDIR is: /
Checking `suspscan'... INFECTED
Searching /tmp for suspicious files
Suspicious: /tmp/.hidden_file

Checking `sniffer'... INFECTED
eth0: Promiscuous mode detected
```

### Output Indicators

| Indicator | Meaning |
|-----------|---------|
| `not found` | Binary not installed |
| `not infected` | Binary is clean |
| `INFECTED` | Suspicious modification detected |
| `unknown` | Cannot determine status |

### False Positives

chkrootkit may report false positives. Before taking action:

1. Read `/usr/share/doc/chkrootkit/README.FALSE-POSITIVES`
2. Verify the finding manually
3. Use additional tools to confirm
4. Consider the system context

---

## Practical Lab Exercises

### Lab 1: Basic System Scan

```bash
# Run full scan
sudo chkrootkit

# Save output to file
sudo chkrootkit > scan_results.txt 2>&1

# Review results
cat scan_results.txt | grep -i "infected\|suspicious"
```

### Lab 2: Quiet Mode Scanning

```bash
# Only show suspicious results
sudo chkrootkit -q

# Save quiet output
sudo chkrootkit -q > suspicious.txt 2>&1
```

### Lab 3: Expert Mode Scanning

```bash
# Thorough scan
sudo chkrootkit -x

# Expert mode with specific tests
sudo chkrootkit -x suspscan netscan
```

### Lab 4: Network Analysis

```bash
# Check for sniffers
sudo chkrootkit sniffer

# Check for suspicious connections
sudo chkrootkit netscan

# Check for shellshock vulnerability
sudo chkrootkit shellshock
```

### Lab 5: Log Analysis

```bash
# Check lastlog for anomalies
sudo chkrootkit chklastlog

# Check wtmp for anomalies
sudo chkrootkit chkwtmp
```

---

## Integration with Other Tools

### chkrootkit + rkhunter

```bash
# Run both for comprehensive coverage
sudo chkrootkit > chkrootkit_results.txt 2>&1
sudo rkhunter --check --sk > rkhunter_results.txt 2>&1

# Compare results
diff chkrootkit_results.txt rkhunter_results.txt
```

### chkrootkit + unhide

```bash
# Check for rootkits
sudo chkrootkit

# Check for hidden processes
sudo unhide proc
sudo unhide sys
```

### chkrootkit + ClamAV

```bash
# Run chkrootkit
sudo chkrootkit

# Scan with ClamAV
sudo clamscan -r / --infected

# Combine results
cat chkrootkit_results.txt clamav_results.txt > combined_scan.txt
```

### chkrootkit + AIDE/Tripwire

```bash
# Run chkrootkit
sudo chkrootkit

# Check file integrity with AIDE
sudo aide --check

# Or Tripwire
sudo tripwire --check
```

### Automated Scanning

```bash
# Set up daily cron job
echo "0 2 * * * root /usr/sbin/chkrootkit > /var/log/chkrootkit.log 2>&1" | sudo tee /etc/cron.d/chkrootkit

# Or use systemd timer
sudo systemctl enable chkrootkit-daily.timer
sudo systemctl start chkrootkit-daily.timer
```

---

## Common Forensic Scenarios

### Scenario 1: Incident Response

```bash
# Initial triage scan
sudo chkrootkit -q > incident_scan.txt 2>&1

# Check for known indicators
grep -i "infected\|suspicious\|backdoor" incident_scan.txt

# Document findings
date > incident_report.txt
echo "Scan results:" >> incident_report.txt
cat incident_scan.txt >> incident_report.txt
```

### Scenario 2: Compromised System Investigation

```bash
# Run in expert mode
sudo chkrootkit -x > compromised_scan.txt 2>&1

# Check specific tests
sudo chkrootkit bindshell
sudo chkrootkit suspscan
sudo chkrootkit sniffer

# Look for modified binaries
sudo chkrootkit ps ls netstat find
```

### Scenario 3: Malware Detection

```bash
# Scan for suspicious files
sudo chkrootkit suspscan > malware_scan.txt 2>&1

# Check for known malware signatures
grep -i "malware\|virus\|trojan" malware_scan.txt

# Check network indicators
sudo chkrootkit netscan >> malware_scan.txt 2>&1
```

### Scenario 4: System Hardening Verification

```bash
# Verify system integrity
sudo chkrootkit > hardening_check.txt 2>&1

# Check for vulnerabilities
sudo chkrootkit shellshock

# Verify no backdoors
sudo chkrootkit bindshell
```

### Scenario 5: Regular Security Monitoring

```bash
# Daily automated scan
sudo chkrootkit -q > /var/log/chkrootkit/$(date +%Y%m%d).log 2>&1

# Alert on suspicious findings
if grep -q "INFECTED" /var/log/chkrootkit/$(date +%Y%m%d).log; then
    echo "ALERT: chkrootkit found suspicious activity" | mail -s "Security Alert" admin@example.com
fi
```

---

## Troubleshooting

### "Permission denied"

```bash
# Run as root
sudo chkrootkit

# Or with specific capabilities
sudo setcap cap_sys_ptrace=eip /usr/sbin/chkrootkit
```

### False Positives

```bash
# Read the false positives guide
cat /usr/share/doc/chkrootkit/README.FALSE-POSITIVES

# Exclude known safe files
sudo chkrootkit -e '/usr/bin/vmstat /usr/bin/lsof /usr/bin/top'
```

### "Command not found"

```bash
# Install chkrootkit
sudo apt install chkrootkit

# Check installation
which chkrootkit
```

### Slow Scanning

```bash
# Run specific tests only
sudo chkrootkit suspscan netscan

# Skip NFS mounts
sudo chkrootkit -n
```

### No Output

```bash
# Run with verbose output
sudo chkrootkit -d

# Check if tests are available
chkrootkit -l
```

---

## Best Practices

1. **Run as root** — chkrootkit needs full system access
2. **Use expert mode** — `-x` for thorough scanning
3. **Check false positives** — read the documentation
4. **Combine with other tools** — don't rely on single scanner
5. **Automate scans** — set up cron jobs for regular monitoring
6. **Save all output** — maintain scan history
7. **Document findings** — note what was found and actions taken
8. **Verify manually** — don't trust automated results blindly
9. **Update regularly** — keep chkrootkit database current
10. **Monitor regularly** — periodic scans catch new infections

---

## Summary

chkrootkit is essential for rootkit detection. Key capabilities:

- Checks for 70+ known rootkits
- Examines system binaries for modifications
- Detects suspicious network activity
- Checks log files for anomalies
- Supports automated scanning

### Quick Reference

```bash
# Full scan
sudo chkrootkit

# Quiet mode (suspicious only)
sudo chkrootkit -q

# Expert mode (thorough)
sudo chkrootkit -x

# Specific test
sudo chkrootkit suspscan

# Show available tests
chkrootkit -l

# Save results
sudo chkrootkit > results.txt 2>&1
```

---

## References

- chkrootkit Homepage: https://www.chkrootkit.org/
- Kali Linux chkrootkit: https://www.kali.org/tools/chkrootkit/
- Linux Rootkit Detection: https://www.sans.org/white-papers/detecting-rootkits/
