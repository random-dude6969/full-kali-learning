# Lynis Cheat Sheet

## Quick Start

```bash
sudo lynis audit system
```

## Primary Commands

| Command | Description |
|---------|-------------|
| `audit system` | Full local security audit |
| `audit system remote <host>` | Remote security scan |
| `audit dockerfile <file>` | Analyze a Dockerfile |
| `show version` | Lynis version |
| `show help` | Help text |
| `show profiles` | Discovered audit profiles |
| `show settings` | Active settings |
| `show options` | All available parameters |
| `update info` | Check for updates |

## Common Options

| Option | Short | Description |
|--------|-------|-------------|
| `--quick` | `-Q` | No user prompts |
| `--quiet` | | Warnings only (includes `--quick`) |
| `--pentest` | | Non-privileged pentest mode |
| `--forensics` | | Forensic analysis mode |
| `--cronjob` | | Automated mode (`-c -Q`) |
| `--auditor "Name"` | | Set auditor name in report |
| `--profile <file>` | | Custom profile |
| `--no-colors` | | Disable colors |
| `--reverse-colors` | | Light background colors |
| `--verbose` | | Extra detail |
| `--debug` | | Debug logging to screen |
| `--no-log` | | Skip log file creation |
| `--version` | `-V` | Version and exit |
| `--man` | | Man page |
| `--wait` | | Wait between test sets |
| `--check-update` | | Check for new version |
| `--plugindir <path>` | | Custom plugin directory |
| `--upload` | | Upload to central node |

## Scan Profiles

### Full Audit

```bash
sudo lynis audit system
```

### Quick Automated Scan

```bash
sudo lynis -Q --cronjob
```

### Pentest Mode (Non-Root)

```bash
lynis --pentest
```

### Forensic Mode

```bash
sudo lynis --forensics
```

### With Auditor Info

```bash
sudo lynis audit system --auditor "Pentest Team" --cronjob
```

### Custom Profile

```bash
sudo lynis audit system --profile /etc/lynis/custom.prf
```

### Skip Specific Tests

```bash
# In profile: test_skip_always=SSH-0280;SSH-0282;FIRE-4510
sudo lynis audit system --profile /etc/lynis/custom.prf
```

## Output Files

| File | Description |
|------|-------------|
| `/var/log/lynis.log` | Detailed scan log (purged each run) |
| `/var/log/lynis-report.dat` | Machine-readable report (key=value) |

## Parsing Report Data

```bash
# All warnings
grep "warning=" /var/log/lynis-report.dat

# All suggestions
grep "suggestion=" /var/log/lynis-report.dat

# Hardening index score
grep "hardening_index" /var/log/lynis-report.dat

# Installed packages
grep "installed_package\[\]" /var/log/lynis-report.dat

# Kernel version
grep "kernel_version" /var/log/lynis-report.dat

# SUID binaries found
grep "suid_bin\[\]" /var/log/lynis-report.dat

# Open ports
grep "netstat_port\[\]" /var/log/lynis-report.dat

# Services
grep "service\[\]" /var/log/lynis-report.dat

# SSL/TLS findings
grep -i "ssl\|tls\|certificate" /var/log/lynis-report.dat

# SSH configuration
grep -i "ssh" /var/log/lynis-report.dat

# Firewall status
grep -i "firewall" /var/log/lynis-report.dat

# Password policy
grep -i "password" /var/log/lynis-report.dat
```

## Comparing Scans

```bash
# Before hardening
sudo lynis audit system --profile /etc/lynis/baseline.prf
cp /var/log/lynis-report.dat /tmp/before.dat

# After hardening
sudo lynis audit system --profile /etc/lynis/baseline.prf
cp /var/log/lynis-report.dat /tmp/after.dat

# Compare
diff /tmp/before.dat /tmp/after.dat

# Compare hardening index
grep "hardening_index" /tmp/before.dat /tmp/after.dat
```

## Compliance Scanning

### PCI DSS

```bash
sudo lynis audit system --profile /etc/lynis/pci-dss.prf
```

### ISO 27001

```bash
sudo lynis audit system --profile /etc/lynis/iso27001.prf
```

### HIPAA

```bash
sudo lynis audit system --profile /etc/lynis/hipaa.prf
```

### CIS Benchmark

```bash
sudo lynis audit system --profile /etc/lynis/cis.prf
```

### Generate Compliance Report

```bash
sudo lynis audit system --auditor "Compliance Team" --cronjob
grep "suggestion\|warning" /var/log/lynis-report.dat | sort > compliance-issues.txt
```

## Automation

### Cron Job (Daily at 2 AM)

```bash
0 2 * * * /usr/sbin/lynis audit system --cronjob --auditor "Automated" >> /var/log/lynis-cron.log 2>&1
```

### CI/CD Gate Script

```bash
#!/bin/bash
lynis audit system --cronjob --quiet --profile /etc/lynis/ci.prf
INDEX=$(grep "hardening_index" /var/log/lynis-report.dat | cut -d= -f2)
if [ "$INDEX" -lt 65 ]; then
    echo "FAIL: Index $INDEX < 65"
    exit 1
fi
echo "PASS: Index $INDEX"
exit 0
```

### Batch Scan Multiple Hosts

```bash
for host in web01 db01 app01; do
    ssh "$host" 'lynis audit system --cronjob --quiet'
    scp "$host:/var/log/lynis-report.dat" "/reports/${host}-report.dat"
done
```

### Automated Hardening Check

```bash
#!/bin/bash
REPORT="/var/log/lynis-report.dat"
sudo lynis audit system --cronjob --quiet

WARNINGS=$(grep -c "warning=" "$REPORT")
SUGGESTIONS=$(grep -c "suggestion=" "$REPORT")
INDEX=$(grep "hardening_index=" "$REPORT" | cut -d= -f2)

echo "=== Lynis Audit Summary ==="
echo "Hardening Index: $INDEX/100"
echo "Warnings: $WARNINGS"
echo "Suggestions: $SUGGESTIONS"
echo ""
echo "Top Warnings:"
grep "warning=" "$REPORT" | head -10
echo ""
echo "Top Suggestions:"
grep "suggestion=" "$REPORT" | head -10
```

## Pentest Mode — What to Look For

### SUID Binaries

```bash
# Lynis finds these; cross-reference with GTFOBins
grep "suid_bin" /var/log/lynis-report.dat
```

### Sudo Misconfigurations

```bash
# Check what Lynis found
grep -i "sudo" /var/log/lynis-report.dat

# Manual check
sudo -l
```

### Cron Job Weaknesses

```bash
# Lynis checks cron; verify manually
crontab -l
ls -al /etc/cron*
cat /etc/crontab
```

### Kernel Exploits

```bash
# Lynis notes kernel version; search for exploits
uname -r
searchsploit "linux kernel $(uname -r | cut -d. -f1,2)"
```

### Container Escape

```bash
# Lynis detects Docker/LXC; check manually
ls -la /var/run/docker.sock
id | grep -i docker
id | grep -i lxd
```

### World-Readable Sensitive Files

```bash
# Lynis flags these; verify
cat /etc/shadow 2>/dev/null
find / -name "*.bak" -o -name "*_history" 2>/dev/null
```

### Weak File Permissions

```bash
# Lynis checks permissions; verify
find /etc -perm -o+w -type f 2>/dev/null
find /usr/local/bin -perm -o+w -type f 2>/dev/null
```

### Network Exposure

```bash
# Lynis checks open ports; verify
ss -tulnp
netstat -tulnp
```

## Custom Profile Template

```bash
# /etc/lynis/custom.prf

# Profile settings
profile_version=20240101
auditor="Security Team"
company="Example Corp"

# Log settings
log_file=/var/log/lynis-custom.log
report_file=/var/log/lynis-custom-report.dat

# Plugin directory
plugin_dir=/etc/lynis/plugins

# Skip specific tests (find IDs in lynis.log)
test_skip_always=SSH-0280;SSH-0282

# Network scan
network_timeout=3

# Compliance mode
compliance_mode=pci-dss
```

## Hardening Index Reference

| Score | Meaning |
|-------|---------|
| 0-30 | Severely vulnerable, immediate action required |
| 31-50 | Significant weaknesses, prioritize fixes |
| 51-65 | Moderate security, address high-priority items |
| 66-80 | Good security posture, minor improvements needed |
| 81-100 | Strong security, maintain and monitor |

## Quick Reference — Output Formats

```bash
# Human-readable with colors
sudo lynis audit system

# Machine-readable (cronjob mode)
sudo lynis --cronjob

# JSON-like parsing
grep "=" /var/log/lynis-report.dat | sort

# CSV export
awk -F= '{print $1","$2}' /var/log/lynis-report.dat > report.csv
```

## Tips

- Always run as root for full audit; use `--pentest` without root
- The log file is purged each scan — back up before re-running
- Use `--cronjob` for automated scanning
- `--quick` skips interactive prompts, essential for scripts
- Custom profiles let you tailor scans to your environment
- The hardening index provides a single metric for tracking progress
- Lynis checks are non-intrusive — safe for production systems
- Use `lynis show options` to see all available parameters
- The report file enables scripted compliance checking
- Combine with LinPEAS for comprehensive privilege escalation assessment
