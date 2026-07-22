# rkhunter — Rootkit Hunter

## Table of Contents

1. [Overview](#overview)
2. [How rkhunter Works](#how-rkhunter-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Understanding the Output](#understanding-the-output)
7. [Configuration](#configuration)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

**Rootkit Hunter (rkhunter)** is a Unix-based tool that scans systems for known and unknown rootkits, backdoors, sniffers, and exploits. It performs several checks including:

- SHA256 hash changes of system binaries
- Files commonly created by rootkits
- Executables with anomalous file permissions
- Suspicious strings in kernel modules
- Hidden files in system directories
- Optionally scanning within files

### Key Features

- Comprehensive rootkit detection
- File integrity checking
- Configuration file checking
- Network port scanning
- Hidden file detection
- Rootkit database with regular updates

### rkhunter vs chkrootkit

| Feature | rkhunter | chkrootkit |
|---------|----------|------------|
| Hash checking | SHA256/SHA1/MD5 | MD5 |
| Rootkit database | Extensive | Extensive |
| File permissions | Yes | Limited |
| Network checks | Yes | Yes |
| Log analysis | Yes | Limited |
| Updates | Manual | Manual |

---

## How rkhunter Works

### Detection Methods

1. **File Hash Comparisons** — Compares system binary hashes against known good values
2. **File Property Checks** — Checks file permissions, sizes, timestamps
3. **Rootkit Signatures** — Scans for known rootkit files and patterns
4. **Hidden Files** — Detects hidden files in system directories
5. **Network Checks** — Identifies suspicious network connections
6. **Kernel Module Analysis** — Checks for suspicious kernel modules

### Rootkit Database

rkhunter uses a database of known rootkit signatures. The database is stored in:

```
/var/lib/rkhunter/db/
```

### Update Process

```bash
# Update rkhunter database
sudo rkhunter --update

# Check for updates
sudo rkhunter --versioncheck
```

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install rkhunter
```

### Verify Installation

```bash
rkhunter --version
```

### Initial Setup

```bash
# Update the database
sudo rkhunter --update

# Set file properties
sudo rkhunter --propupd
```

---

## Command Syntax

```bash
rkhunter [options]
```

### Basic Usage

```bash
# Run a full system check
sudo rkhunter --check

# Run with skip keypress
sudo rkhunter --check --sk

# Update database
sudo rkhunter --update

# Update file properties
sudo rkhunter --propupd
```

---

## Options Reference

### Main Operations

| Option | Description |
|--------|-------------|
| `-c, --check` | Check the local system |
| `--update` | Check for database updates |
| `--versioncheck` | Check for rkhunter version updates |
| `--propupd` | Update file properties database |
| `--unlock` | Remove the lock file |
| `--list` | List available tests, languages, rootkits, etc. |
| `--config-check` | Check configuration files |

### Check Options

| Option | Description |
|--------|-------------|
| `--sk, --skip-keypress` | Don't wait for keypress after each test |
| `--cronjob` | Run as cron job (implies -c, --sk, --nocolors) |
| `--disable test[,test]` | Disable specific tests |
| `--enable test[,test]` | Enable specific tests |
| `--hash {MD5\|SHA1\|SHA256\|...}` | Hash function (default SHA256) |
| `--summary` | Show summary (default) |
| `--nosummary` | Don't show summary |
| `--rwo, --report-warnings-only` | Show only warnings |

### Output Options

| Option | Description |
|--------|-------------|
| `-l, --logfile [file]` | Write to logfile (default: /var/log/rkhunter.log) |
| `--append-log` | Append to logfile |
| `--noappend-log` | Overwrite logfile |
| `--nolog` | Don't write to logfile |
| `--nocolors` | Black and white output |
| `--display-logfile` | Display logfile at end |
| `-q, --quiet` | Quiet mode (no output) |

### Configuration Options

| Option | Description |
|--------|-------------|
| `--configfile file` | Use specified config file |
| `--dbdir directory` | Use specified database directory |
| `--tmpdir directory` | Use specified temp directory |
| `--bindir directory` | Use specified command directories |
| `--pkgmgr {RPM\|DPKG\|...}` | Package manager for file properties |

---

## Understanding the Output

### Check Summary

```
[ OK ] MD5 hash: /usr/bin/awk
[ OK ] MD5 hash: /usr/bin/basename
[ WARNING ] File is empty: /var/log/lastlog
[ INFECTED ] Hidden file: /usr/share/.hidden_rootkit
```

### Output Indicators

| Indicator | Meaning |
|-----------|---------|
| `[ OK ]` | Check passed |
| `[ WARNING ]` | Potential issue (may be false positive) |
| `[ INFECTED ]` | Suspicious finding detected |
| `[ NOT FOUND ]` | Expected file not found |
| `[ SKIPPED ]` | Test was skipped |

### Log File Location

```
/var/log/rkhunter.log
```

### Rootkit Detection

```
[ INFECTED ] Rootkit found: Linux/Xor.DDoS
[ WARNING ] Possible rootkit: /usr/lib/libamplify.so
```

---

## Configuration

### Main Configuration File

```
/etc/rkhunter.conf
```

### Key Configuration Options

```bash
# Log file location
LOGFILE=/var/log/rkhunter.log

# Rootkit database directory
DBDIR=/var/lib/rkhunter/db

# Temporary directory
TMPDIR=/var/tmp

# Email alerts
MAIL-ON-WARNING=root@localhost

# Scan modes
SCAN_MODES=Defaults
```

### Enabling/Disabling Tests

```bash
# In /etc/rkhunter.conf:
# Disable specific tests
disable_tests=suspscan hidden_procs

# Enable specific tests
enable_tests=deleted_files
```

### Package Manager Configuration

```bash
# For Debian/Ubuntu
PKGDMGR=DPKG

# For RPM-based
PKGDMGR=RPM
```

---

## Practical Lab Exercises

### Lab 1: Basic System Check

```bash
# Run full system check
sudo rkhunter --check

# Skip keypress for automation
sudo rkhunter --check --sk

# Save results to file
sudo rkhunter --check --sk -l /tmp/rkhunter_results.txt
```

### Lab 2: Update and Check

```bash
# Update database first
sudo rkhunter --update

# Update file properties
sudo rkhunter --propupd

# Run check
sudo rkhunter --check --sk
```

### Lab 3: Automated Scanning

```bash
# Set up daily cron job
echo "0 2 * * * root /usr/bin/rkhunter --check --sk --append-log --nocolors" | sudo tee /etc/cron.d/rkhunter

# Or use systemd timer
sudo systemctl enable rkhunter.timer
sudo systemctl start rkhunter.timer
```

### Lab 4: Specific Test Selection

```bash
# Run only specific tests
sudo rkhunter --check --sk --enable group_files,hidden_procs

# Disable specific tests
sudo rkhunter --check --sk --disable suspscan
```

### Lab 5: Log Analysis

```bash
# View the log
cat /var/log/rkhunter.log

# Search for warnings
grep "WARNING\|INFECTED" /var/log/rkhunter.log

# View scan summary
tail -50 /var/log/rkhunter.log
```

---

## Integration with Other Tools

### rkhunter + chkrootkit

```bash
# Run both for comprehensive coverage
sudo rkhunter --check --sk > rkhunter_results.txt 2>&1
sudo chkrootkit > chkrootkit_results.txt 2>&1

# Compare findings
diff rkhunter_results.txt chkrootkit_results.txt
```

### rkhunter + unhide

```bash
# Check for rootkits
sudo rkhunter --check --sk

# Check for hidden processes
sudo unhide proc
sudo unhide sys
```

### rkhunter + AIDE/Tripwire

```bash
# rkhunter for rootkit detection
sudo rkhunter --check --sk

# AIDE for file integrity
sudo aide --check

# Tripwire for policy checking
sudo tripwire --check
```

### rkhunter + ClamAV

```bash
# rkhunter for rootkits
sudo rkhunter --check --sk

# ClamAV for malware
sudo clamscan -r / --infected
```

### rkhunter + Log Analysis

```bash
# Parse rkhunter logs
grep "INFECTED" /var/log/rkhunter.log | awk '{print $1, $2, $NF}'

# Send email alerts
cat /var/log/rkhunter.log | mail -s "rkhunter Alert" admin@example.com
```

---

## Common Forensic Scenarios

### Scenario 1: Incident Response

```bash
# Initial triage
sudo rkhunter --check --sk --nocolors > incident_scan.txt 2>&1

# Check for rootkits
grep "INFECTED" incident_scan.txt

# Document findings
date > incident_report.txt
echo "rkhunter Scan Results:" >> incident_report.txt
cat incident_scan.txt >> incident_report.txt
```

### Scenario 2: Compromised System Investigation

```bash
# Full check with all tests
sudo rkhunter --check --sk --enable all

# Check specific areas
sudo rkhunter --check --sk --enable hidden_procs,network
```

### Scenario 3: Malware Detection

```bash
# Check for known malware
sudo rkhunter --check --sk --enable malware_scan

# Check for suspicious files
sudo rkhunter --check --sk --suspscan
```

### Scenario 4: System Hardening Verification

```bash
# Verify system integrity
sudo rkhunter --check --sk --pkgmgr DPKG

# Check file properties
sudo rkhunter --propupd
```

### Scenario 5: Regular Monitoring

```bash
# Automated daily scan
sudo rkhunter --check --sk --append-log --nocolors

# Weekly full scan
echo "0 3 * * 0 root /usr/bin/rkhunter --check --sk --display-logfile" | sudo tee /etc/cron.d/rkhunter_weekly
```

---

## Troubleshooting

### "Lock file exists"

```bash
# Remove the lock file
sudo rkhunter --unlock

# Or manually remove
sudo rm /var/run/rkhunter.pid
```

### "Database not updated"

```bash
# Update manually
sudo rkhunter --update

# Check internet connectivity
ping -c 3 google.com

# Force update
sudo rkhunter --update --nocolors
```

### False Positives

```bash
# Add to whitelist in /etc/rkhunter.conf:
# SCRIPTWHITELIST=/path/to/safe/script
# IMPL_WHITELIST=/path/to/safe/library

# Or disable specific tests
sudo rkhunter --check --sk --disable suspscan
```

### "Command not found"

```bash
# Install rkhunter
sudo apt install rkhunter

# Check installation
which rkhunter
```

### Slow Scanning

```bash
# Skip certain tests
sudo rkhunter --check --sk --disable suspscan

# Use quiet mode
sudo rkhunter --check --sk -q
```

---

## Best Practices

1. **Update before scanning** — `rkhunter --update`
2. **Update file properties** — `rkhunter --propupd`
3. **Run as root** — full system access required
4. **Use --sk for automation** — skip keypress
5. **Check logs regularly** — `/var/log/rkhunter.log`
6. **Set up automated scanning** — cron jobs or systemd timers
7. **Combine with other tools** — don't rely on single scanner
8. **Whitelist known safe items** — reduce false positives
9. **Document findings** — save and archive scan results
10. **Verify manually** — don't trust automated results blindly

---

## Summary

rkhunter is essential for rootkit detection. Key capabilities:

- Detects known and unknown rootkits
- Checks file integrity with SHA256 hashing
- Scans for suspicious network activity
- Detects hidden files and processes
- Supports automated scanning

### Quick Reference

```bash
# Full system check
sudo rkhunter --check

# Skip keypress
sudo rkhunter --check --sk

# Update database
sudo rkhunter --update

# Update file properties
sudo rkhunter --propupd

# Check version
rkhunter --version

# Remove lock file
sudo rkhunter --unlock
```

---

## References

- rkhunter Homepage: https://rkhunter.sourceforge.net/
- Kali Linux rkhunter: https://www.kali.org/tools/rkhunter/
- Linux Rootkit Detection: https://www.sans.org/white-papers/detecting-rootkits/
