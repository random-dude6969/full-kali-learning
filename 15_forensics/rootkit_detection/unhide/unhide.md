# unhide — Detect Hidden Processes

## Table of Contents

1. [Overview](#overview)
2. [How unhide Works](#how-unhide-works)
3. [Installation](#installation)
4. [Command Syntax](#command-syntax)
5. [Options Reference](#options-reference)
6. [Detection Techniques](#detection-techniques)
7. [Understanding the Output](#understanding-the-output)
8. [Practical Lab Exercises](#practical-lab-exercises)
9. [Integration with Other Tools](#integration-with-other-tools)
10. [Common Forensic Scenarios](#common-forensic-scenarios)
11. [Troubleshooting](#troubleshooting)
12. [Best Practices](#best-practices)
13. [Summary](#summary)

---

## Overview

**unhide** is a forensic tool that finds processes and TCP/UDP ports hidden by rootkits, Linux kernel modules, or other techniques. It includes two main utilities:

- **unhide** — Detects hidden processes
- **unhide-tcp** — Detects hidden TCP/UDP ports

Rootkits often hide malicious processes and open ports from system tools like `ps`, `netstat`, and `lsof`. unhide uses multiple detection techniques to identify these hidden entities.

### Key Features

- Six different process detection techniques
- TCP/UDP port detection
- Cross-references multiple system sources
- Detects kernel-level hiding
- Fast and lightweight

---

## How unhide Works

### Process Detection Techniques

1. **Compare /proc vs /bin/ps** — Cross-references procfs with ps output
2. **Compare /bin/ps with procfs walking** — Walks procfs directly
3. **Compare /bin/ps with syscalls** — Uses system calls directly
4. **PID bruteforcing** — Tries all possible PIDs
5. **Reverse search** — Verifies all threads seen by ps are seen by kernel
6. **Quick compare** — Fast combination of multiple methods

### Port Detection Technique

unhide-tcp identifies TCP/UDP ports that are listening but not listed in `/bin/netstat` through **brute forcing** of all available ports.

---

## Installation

### On Kali Linux

```bash
sudo apt update
sudo apt install unhide
```

### Verify Installation

```bash
unhide -V
```

---

## Command Syntax

### unhide (Process Detection)

```bash
unhide [options] test_list
```

### unhide-tcp (Port Detection)

```bash
unhide-tcp [options]
```

---

## Options Reference

### unhide Options

| Option | Description |
|--------|-------------|
| `-V` | Show version |
| `-v` | Verbose mode |
| `-h` | Help |
| `-m` | More checks (procfs, checkopendir, checkchdir) |
| `-r` | Use alternate sysinfo test |
| `-f` | Log to unhide-linux.log |
| `-o` | Same as -f |
| `-d` | Double check in brute test |
| `-u` | Inhibit stdout buffering |

### Test Lists

| Test | Description |
|------|-------------|
| `brute` | PID bruteforcing |
| `proc` | Compare /proc vs /bin/ps |
| `procall` | Combine proc and procfs |
| `procfs` | Compare ps with procfs walking |
| `quick` | Quick compare (fast, may have false positives) |
| `reverse` | Verify all threads are seen |
| `sys` | Compare ps with syscalls |

### Elementary Tests

| Test | Description |
|------|-------------|
| `checkbrute` | PID bruteforcing |
| `checkchdir` | Compare ps with chdir in procfs |
| `checkgetaffinity` | Check sched_getaffinity |
| `checkgetparam` | Check sched_getparam |
| `checkgetpgid` | Check getpgid |
| `checkgetprio` | Check getpriority |
| `checkRRgetinterval` | Check sched_rr_get_interval |
| `checkgetsched` | Check sched_getscheduler |
| `checkgetsid` | Check getsid |
| `checkkill` | Check kill system call |
| `checknoprocps` | Compare syscalls without ps |
| `checkopendir` | Compare ps with opendir in procfs |
| `checkproc` | Compare /proc vs /bin/ps |
| `checkquick` | Quick compare |
| `checkreaddir` | Compare ps with readdir |
| `checkreverse` | Verify all threads |
| `checksysinfo` | Compare ps with sysinfo |
| `checksysinfo2` | Alternate sysinfo test |
| `checksysinfo3` | Third sysinfo test |

### unhide-tcp Options

| Option | Description |
|--------|-------------|
| `-V` | Show version |
| `-v` | Verbose mode |
| `-h` | Help |
| `-f` | Show fuser output for hidden ports |
| `-l` | Show lsof output for hidden ports |
| `-o` | Log to unhide-tcp.log |
| `-s` | Quick version for servers with many ports |
| `-n` | Use netstat instead of ss |

---

## Detection Techniques

### Technique 1: /proc vs /bin/ps

```bash
unhide proc
```

Compares the list of processes from `/proc` with the output of `/bin/ps`. If a process appears in `/proc` but not in `ps`, it may be hidden.

### Technique 2: procfs Walking

```bash
unhide procfs
```

Walks the `/proc` filesystem directly, comparing with `ps` output.

### Technique 3: Syscall Scanning

```bash
unhide sys
```

Uses system calls (`getpgid`, `getsid`, `kill`, etc.) to detect processes that may be hidden from `/proc` and `ps`.

### Technique 4: PID Bruteforcing

```bash
unhide brute
```

Tries every possible PID (1-32768 or higher) and checks if a process exists at each PID using system calls.

### Technique 5: Reverse Search

```bash
unhide reverse
```

Verifies that all processes shown by `ps` are also visible in `/proc` and via system calls. Detects if a rootkit is replacing real processes with fake ones.

### Technique 6: Quick Scan

```bash
unhide quick
```

Combines proc, procfs, and sys techniques in a quick scan. About 20 times faster but may have more false positives.

---

## Understanding the Output

### Clean Output

```
Unhide 20240509
Copyright © 2010-2024 Yago Jesus & Patrick Gouin

[*] Starting check using /proc
[*] Starting check using syscalls
[*] Starting check using /bin/ps

No hidden processes found
```

### Hidden Process Found

```
[*] Starting check using /proc
[*] Starting check using syscalls
[*] Starting check using /bin/ps

[!] Hidden process found: PID 1234
[!] Hidden process found: PID 5678

2 hidden processes found
```

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | No hidden processes found |
| `1` | Hidden or fake thread found |

---

## Practical Lab Exercises

### Lab 1: Basic Process Scan

```bash
# Run standard tests
sudo unhide proc sys

# Quick scan
sudo unhide quick

# Save results
sudo unhide proc sys > unhide_results.txt 2>&1
```

### Lab 2: Comprehensive Scan

```bash
# Run all tests
sudo unhide -m -d proc procfs sys brute reverse

# With verbose output
sudo unhide -v proc procfs sys brute reverse
```

### Lab 3: Port Detection

```bash
# Check for hidden ports
sudo unhide-tcp

# With verbose output
sudo unhide-tcp -v

# Show fuser/lsof output
sudo unhide-tcp -f -l
```

### Lab 4: Combining Process and Port Detection

```bash
# Check both processes and ports
sudo unhide proc sys > processes.txt 2>&1
sudo unhide-tcp > ports.txt 2>&1

# Review findings
cat processes.txt
cat ports.txt
```

### Lab 5: Automated Scanning

```bash
# Set up automated scan
echo "0 3 * * * root /usr/bin/unhide proc sys > /var/log/unhide.log 2>&1" | sudo tee /etc/cron.d/unhide

# Or with rkhunter integration
echo "0 2 * * * root /usr/bin/unhide proc sys" | sudo tee -a /etc/cron.d/rkhunter
```

---

## Integration with Other Tools

### unhide + rkhunter

```bash
# rkhunter can use unhide for hidden process detection
# Configure in /etc/rkhunter.conf:
# SCRIPTWHITELIST=/usr/bin/unhide

# Run both
sudo rkhunter --check --sk
sudo unhide proc sys
```

### unhide + chkrootkit

```bash
# Run both for comprehensive coverage
sudo chkrootkit
sudo unhide proc sys
sudo unhide-tcp
```

### unhide + ps

```bash
# Compare unhide with ps output
ps aux | wc -l
sudo unhide proc | grep "Hidden process"
```

### unhide + netstat

```bash
# Compare unhide-tcp with netstat
netstat -tlnp | wc -l
sudo unhide-tcp -v
```

### unhide + System Monitoring

```bash
# Use in monitoring scripts
if sudo unhide proc sys 2>&1 | grep -q "Hidden"; then
    echo "ALERT: Hidden processes detected!"
    # Send alert
fi
```

---

## Common Forensic Scenarios

### Scenario 1: Incident Response

```bash
# Initial triage
sudo unhide proc sys > incident_unhide.txt 2>&1
sudo unhide-tcp > incident_ports.txt 2>&1

# Check for hidden processes
grep "Hidden process" incident_unhide.txt

# Check for hidden ports
grep "Hidden" incident_ports.txt
```

### Scenario 2: Malware Detection

```bash
# Check for rootkit hiding
sudo unhide -v proc procfs sys brute reverse

# Check for hidden ports (C2 communication)
sudo unhide-tcp -v -f -l
```

### Scenario 3: Compromised System Investigation

```bash
# Comprehensive scan
sudo unhide -m -d proc procfs sys brute reverse quick

# Document findings
date > investigation.txt
sudo unhide proc sys >> investigation.txt 2>&1
sudo unhide-tcp >> investigation.txt 2>&1
```

### Scenario 4: System Hardening Verification

```bash
# Verify no hidden processes
sudo unhide proc sys

# Verify no hidden ports
sudo unhide-tcp

# Check against baseline
diff unhide_baseline.txt current_scan.txt
```

### Scenario 5: Regular Monitoring

```bash
# Daily automated scan
sudo unhide proc sys > /var/log/unhide/$(date +%Y%m%d).log 2>&1

# Alert on findings
if grep -q "Hidden" /var/log/unhide/$(date +%Y%m%d).log; then
    echo "ALERT: Hidden processes/ports detected" | mail -s "Security Alert" admin@example.com
fi
```

---

## Troubleshooting

### "Command not found"

```bash
# Install unhide
sudo apt install unhide

# Check installation
which unhide unhide-tcp
```

### False Positives

```bash
# Use more tests for confirmation
sudo unhide -m proc procfs sys

# Check with verbose output
sudo unhide -v proc

# Verify manually
ls -la /proc/<PID>
cat /proc/<PID>/status
```

### "Permission denied"

```bash
# Run as root
sudo unhide proc sys
sudo unhide-tcp
```

### Slow Scanning

```bash
# Use quick mode
sudo unhide quick

# Or skip brute test
sudo unhide proc sys reverse
```

### Port Detection Issues

```bash
# Use quick mode for servers with many ports
sudo unhide-tcp -s

# Try netstat instead of ss
sudo unhide-tcp -n
```

---

## Best Practices

1. **Run as root** — full system access required
2. **Use multiple tests** — don't rely on single technique
3. **Combine with other tools** — rkhunter, chkrootkit
4. **Check both processes AND ports** — unhide + unhide-tcp
5. **Use verbose mode** — `-v` for detailed output
6. **Automate scanning** — cron jobs for regular monitoring
7. **Save all output** — maintain scan history
8. **Verify manually** — don't trust automated results blindly
9. **Document findings** — note what was found and actions taken
10. **Update regularly** — keep the tool current

---

## Summary

unhide is essential for detecting hidden processes and ports. Key capabilities:

- Six process detection techniques
- TCP/UDP port detection
- Cross-references multiple system sources
- Detects kernel-level hiding
- Fast and lightweight

### Quick Reference

```bash
# Process detection
sudo unhide proc sys

# Quick scan
sudo unhide quick

# Comprehensive scan
sudo unhide -m -d proc procfs sys brute reverse

# Port detection
sudo unhide-tcp

# Verbose output
sudo unhide -v proc sys

# Log to file
sudo unhide proc sys -f
```

---

## References

- unhide Homepage: https://www.unhide-forensics.info
- Kali Linux unhide: https://www.kali.org/tools/unhide/
- Linux Process Hiding Techniques: https://www.sans.org/white-papers/detecting-rootkits/
