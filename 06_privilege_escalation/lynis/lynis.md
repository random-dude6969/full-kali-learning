---
title: Lynis
tool: lynis
version: 3.1.7
category: Privilege Escalation
platform: Linux/macOS/BSD/Unix
language: Shell (Bash)
license: GPL-3.0
github: https://github.com/CISOfy/lynis
kali: https://www.kali.org/tools/lynis/
mitre_tactic: privilege_escalation
mitre_tactic_id: TA0004
tags:
  - security-audit
  - hardening
  - compliance
  - misconfiguration
  - vulnerability-detection
  - system-audit
  - pentest
  - forensic
---

# Lynis

## 1. Overview

**Lynis** is an open-source security auditing tool for Unix-based systems (Linux, macOS, BSD). It performs in-depth security scans to test system defenses, identify misconfigurations, and provide hardening recommendations. Lynis is used by system administrators, auditors, and penetration testers to discover privilege escalation paths and compliance gaps.

**Key capabilities:**
- In-depth security audit across 80+ test categories
- Compliance testing (ISO 27001, PCI DSS, HIPAA, SOX, CIS)
- Non-privileged pentest mode (`--pentest`)
- Forensic mode for mounted/running systems
- Custom test and plugin support
- Cronjob-compatible output for automation
- HostID fingerprinting for system tracking
- Enterprise integration with central reporting
- Dockerfile analysis support

**Awards and recognition:**
- Best of Open Source Software Awards (2015, 2016)
- ToolsWatch Top Security Tools (2013-2016)
- CII Best Practices badge participant
- Used by thousands of organizations worldwide

**Typical scenarios:** Pre-engagement system hardening assessment, post-exploitation privilege escalation discovery, compliance auditing, security baseline establishment, Dockerfile security review.

## 2. Installation

### Kali Linux (Package)

```bash
sudo apt install lynis
```

**Installed size:** ~1.68 MB. Dependency: `e2fsprogs`.

### From Git (Latest)

```bash
git clone https://github.com/CISOfy/lynis
cd lynis
sudo chown -R 0:0 .
./lynis audit system
```

**Explanation:** No compilation required. The `chown` step is recommended when running as root to avoid file ownership warnings.

### From Source Tarball

```bash
wget https://cisofy.com/files/lynis-latest.tar.gz
tar xzf lynis-latest.tar.gz
cd lynis
./lynis audit system
```

### CISOfy Repository (RPM/DEB)

```bash
# Debian/Ubuntu
echo "deb https://packages.cisofy.com/lynis/cisofy/lynis binary/" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C80E383C3DE5FDC8
sudo apt update && sudo apt install lynis
```

**Caveat:** Distribution packages may lag behind the latest release. For the most current tests and features, use the Git or CISOfy repository.

### Verification

```bash
# Check version
lynis show version

# Check for updates
lynis --check-update

# Verify package integrity (for tarball downloads)
sha256sum lynis-*.tar.gz
openssl sha256 lynis-*.tar.gz
```

## 3. Basic Usage

### First-Time Scan

```bash
sudo lynis audit system
```

**Explanation:** Runs a full security audit with all test categories. Results are displayed on screen and logged to `/var/log/lynis.log`. The report file is `/var/log/lynis-report.dat`.

### Quick Scan (Non-Interactive)

```bash
sudo lynis -Q --cronjob
```

**Explanation:** `-Q` (quiet/quick) suppresses user prompts. `--cronjob` combines `-c -Q` for automated execution. Output is suitable for cron jobs.

### Pentest Mode (Non-Privileged)

```bash
lynis --pentest
```

**Explanation:** Runs without root privileges, showing points of interest for penetration testing. Skips tests requiring elevated permissions but still identifies misconfigurations.

### Forensic Mode

```bash
sudo lynis --forensics
```

**Explanation:** Performs forensics on a running or mounted system. Useful for analyzing disk images or systems in a compromised state.

### Dockerfile Analysis

```bash
lynis audit dockerfile /path/to/Dockerfile
```

**Explanation:** Analyzes a Dockerfile for security misconfigurations, best practice violations, and potential privilege escalation vectors.

## 4. Commands and Options

### Primary Commands

| Command | Description |
|---------|-------------|
| `audit system` | Perform local security scan |
| `audit system remote <host>` | Remote security scan |
| `audit dockerfile <file>` | Analyze a Dockerfile |
| `show version` | Display Lynis version |
| `show help` | Show help |
| `show profiles` | List discovered audit profiles |
| `show settings` | Show active settings |
| `show options` | List all available parameters |
| `show hostids` | Display system identifiers |
| `update info` | Check for updates |

### Common Options

| Option | Short | Description |
|--------|-------|-------------|
| `--quick` | `-Q` | Don't wait for user input |
| `--quiet` | | Only show warnings (includes `--quick`) |
| `--pentest` | | Non-privileged pentest mode |
| `--forensics` | | Forensic mode |
| `--cronjob` | | Automated mode (includes `-c -Q`) |
| `--auditor "Name"` | | Assign auditor name to report |
| `--profile <file>` | | Use custom profile |
| `--no-colors` | | Disable color output |
| `--reverse-colors` | | Optimize for light backgrounds |
| `--verbose` | | Show more details |
| `--debug` | | Debug logging to screen |
| `--no-log` | | Don't create log file |
| `--version` | `-V` | Display version and quit |
| `--man` | | View man page |
| `--wait` | | Wait between test sets |
| `--check-update` | | Check for Lynis updates |
| `--plugindir <path>` | | Define plugin directory |
| `--upload` | | Upload data to central node (Enterprise) |
| `--slow-warning <sec>` | | Threshold for slow test warning (default 10) |

### Complete Scan with Auditor Info

```bash
sudo lynis audit system --auditor "PenTest Team" --cronjob
```

### Scan with Custom Profile

```bash
sudo lynis audit system --profile /etc/lynis/custom.prf
```

### Disable Specific Tests

```bash
# In custom profile (/etc/lynis/custom.prf)
test_skip_always=SSH-0280;SSH-0282;FIRE-4510
```

**Explanation:** Each test has a numeric ID visible in the log file. Skipping tests is useful when false positives occur or when the test doesn't apply.

### View All Options

```bash
lynis show options
```

## 5. Profiles and Configuration

### Default Profile Location

```bash
/etc/lynis/default.prf
# or
./default.prf  (in the lynis directory)
```

### Creating a Custom Profile

```bash
cp /etc/lynis/default.prf /etc/lynis/custom.prf
chmod 600 /etc/lynis/custom.prf
```

### Profile Options

```bash
# Set custom log file
log_file=/var/log/lynis-custom.log

# Set custom report file
report_file=/var/log/lynis-custom-report.dat

# Set plugins directory
plugin_dir=/etc/lynis/plugins

# Skip specific tests
test_skip_always=TEST_ID_1;TEST_ID_2

# Set audit group
audit_group="production-servers"

# Enable specific plugins
plugin=my_custom_plugin

# Set company name
company_name="Example Corp"

# Set auditor name
auditor="Security Team"

# Set custom tests directory
tests_dir=/etc/lynis/custom-tests
```

### HostID Override

```bash
lynis configure settings hostid=$(head -c 64 /dev/random | sha1sum | awk '{print $1}'):hostid2=$(head -c 64 /dev/random | sha256sum | awk '{print $1}')
```

**Explanation:** `hostid` (40 chars) is based on MAC address. `hostid2` (64 chars) is based on public SSH key. Override only when systems are short-lived or auto-generated.

### Profile per System Type

```bash
# Web server profile
cp /etc/lynis/default.prf /etc/lynis/webserver.prf
# Add: tests_web=enabled

# Database server profile
cp /etc/lynis/default.prf /etc/lynis/dbserver.prf
# Add: tests_database=enabled

# High-security profile
cp /etc/lynis/default.prf /etc/lynis/high-security.prf
# Add: strict_mode=enabled
```

## 6. Output and Reporting

### Screen Output

Lynis displays test results in real-time:
- `[ OK ]` — Expected result (good)
- `[ WARNING ]` — Unexpected result (investigate)
- `[ SUGGESTION ]` — Improvement recommendation
- `[ INFO ]` — Informational message

**Caveat:** `[ OK ]` does NOT always mean the system is secure. A test may pass even with a weak configuration. Always review suggestions.

### Log File

```bash
/var/log/lynis.log
```

Contains detailed debugging information, test execution reasons, and suggestions. Purged on each scan — back up before re-running if needed.

### Report File

```bash
/var/log/lynis-report.dat
```

Machine-readable format with key-value pairs:

```
# Remarks
[plugindir]
installed_package[]=lynis-3.1.7
kernel_version=6.1.0-kali9-amd64
```

Use this for scripted analysis and comparison between scans.

### Parsing the Report

```bash
# Extract all warnings
grep "warning=" /var/log/lynis-report.dat

# Extract all suggestions
grep "suggestion=" /var/log/lynis-report.dat

# Extract hardening index
grep "hardening_index" /var/log/lynis-report.dat

# Extract installed packages
grep "installed_package\[\]" /var/log/lynis-report.dat

# Compare two scans
diff /var/log/lynis-scan1.dat /var/log/lynis-scan2.dat
```

### Report File Format

The report file uses a simple key=value format:

```
# Section markers
[plugindir]

# Single values
kernel_version=5.15.0-kali3-amd64
os_name=Kali

# Array values
installed_package[]=lynis-3.1.7
installed_package[]=nmap-7.92
suid_binary[]=/usr/bin/sudo
suid_binary[]=/usr/bin/su

# Warnings and suggestions
warning[]=SSH-0280
suggestion[]=FIRE-4510
```

## 7. Compliance Testing

### PCI DSS Scan

```bash
sudo lynis audit system --profile /etc/lynis/pci-dss.prf
```

### ISO 27001 Scan

```bash
sudo lynis audit system --profile /etc/lynis/iso27001.prf
```

### HIPAA Scan

```bash
sudo lynis audit system --profile /etc/lynis/hipaa.prf
```

### Custom Compliance Baseline

```bash
# Create baseline
sudo lynis audit system --profile /etc/lynis/baseline-prod.prf

# Compare against baseline
grep "hardening_index" /var/log/lynis-report.dat
```

### Generating Compliance Reports

```bash
# Run with auditor identification
sudo lynis audit system \
  --auditor "Security Team" \
  --cronjob \
  --profile /etc/lynis/compliance.prf

# Parse results
grep "suggestion\|warning" /var/log/lynis-report.dat | sort
```

### Compliance Checklist Output

```bash
# Generate checklist-style output
sudo lynis audit system --cronjob
awk -F= '/^warning=/{print "FAIL: " $2}' /var/log/lynis-report.dat
awk -F= '/^suggestion=/{print "WARN: " $2}' /var/log/lynis-report.dat
echo "PASS: $(grep -c '^\[ OK \]' /var/log/lynis.log)"
```

## 8. Pentest Mode (Privilege Escalation Discovery)

### Running Pentest Mode

```bash
lynis --pentest
```

**Explanation:** Non-privileged mode that focuses on finding privilege escalation paths. Skips tests requiring root but highlights:

- SUID/SGID binaries with known privesc vectors
- Writable files in PATH
- Sudo misconfigurations
- Cron job weaknesses
- World-readable sensitive files
- Weak file permissions
- Kernel exploit opportunities
- Docker/LXC escape vectors

### Pentest-Specific Findings

Lynis pentest mode highlights:

| Category | What It Finds |
|----------|---------------|
| SUID/SGID | Known exploitable binaries (GTFOBins) |
| Sudo | NOPASSWD commands, command aliases |
| Cron | Writable scripts, wildcard injection |
| File permissions | World-writable files, weak ownership |
| Kernel | Version check against known exploits |
| Containers | Docker socket access, LXD group membership |
| Network | Exposed services, default credentials |

### Combining with LinPEAS

```bash
# Lynis for overall assessment
lynis --pentest -Q 2>&1 | tee lynis-out.txt

# LinPEAS for deep-dive
./linpeas.sh -a 2>&1 | tee linpeas-out.txt

# Compare findings
grep -i "warning\|suggestion\|suid\|cron\|sudo" lynis-out.txt
```

### Pentest Mode Output Analysis

```bash
# Run pentest mode and capture output
lynis --pentest -Q 2>&1 | tee /tmp/lynis-pentest.txt

# Extract SUID findings
grep -i "suid\|sgid" /tmp/lynis-pentest.txt

# Extract cron findings
grep -i "cron\|scheduled" /tmp/lynis-pentest.txt

# Extract sudo findings
grep -i "sudo\|privilege" /tmp/lynis-pentest.txt

# Extract file permission findings
grep -i "permission\|writable\|world" /tmp/lynis-pentest.txt

# Extract kernel findings
grep -i "kernel\|exploit\|version" /tmp/lynis-pentest.txt

# Extract container findings
grep -i "docker\|container\|lxc\|lxd" /tmp/lynis-pentest.txt
```

## 9. Automation and Scheduling

### Cron Job Setup

```bash
# Daily audit at 2 AM
0 2 * * * /usr/sbin/lynis audit system --cronjob --auditor "Automated" >> /var/log/lynis-cron.log 2>&1
```

### CI/CD Integration

```bash
#!/bin/bash
# security-gate.sh
lynis audit system --cronjob --quiet --profile /etc/lynis/ci.prf

INDEX=$(grep "hardening_index" /var/log/lynis-report.dat | cut -d= -f2)
WARNINGS=$(grep -c "warning" /var/log/lynis-report.dat)

if [ "$INDEX" -lt 65 ]; then
    echo "FAIL: Hardening index $INDEX below threshold 65"
    exit 1
fi

echo "PASS: Hardening index $INDEX with $WARNINGS warnings"
exit 0
```

### Batch Scanning Multiple Hosts

```bash
#!/bin/bash
# scan-all.sh
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

### Scheduled Compliance Monitoring

```bash
# Weekly compliance scan
0 3 * * 0 /usr/sbin/lynis audit system --cronjob --profile /etc/lynis/compliance.prf --auditor "Compliance Bot"

# Generate weekly report
0 4 * * 0 /usr/bin/lynis-audit-report.sh
```

## 10. Plugins and Custom Tests

### Plugin Directory

```bash
/usr/local/lynis/plugins
/usr/local/share/lynis/plugins
/usr/share/lynis/plugins
/etc/lynis/plugins
```

### Enabling Plugins

```bash
# In profile
plugin=my_custom_plugin

# Or via command line
lynis audit system --plugindir /path/to/plugins
```

### Plugin Phases

- **Phase 1:** Initialize and gather data
- **Phase 2:** Process results and correlate data

### Custom Test File

```bash
# Create custom test file
touch /etc/lynis/tests_custom
chmod 700 /etc/lynis/tests_custom
```

### Writing Custom Tests

```bash
#!/bin/bash
# /etc/lynis/tests_custom

# Test: Check for world-readable SSH keys
if [ -r /home/*/.ssh/id_rsa ]; then
    Register --test-number 99999
    LogText "RESULT: World-readable SSH key found"
    IncrementTestsFailed
else
    Register --test-number 99999
    LogText "RESULT: No world-readable SSH keys found"
    IncrementTestsPassed
fi
```

## 11. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Lynis Check |
|-------------|----------------|-------------|
| T1068 | Exploitation for Privilege Escalation | Kernel version, SUID binaries, capabilities |
| T1078 | Valid Accounts | Password policies, account configurations |
| T1082 | System Information Discovery | OS info, hardware, network configuration |
| T1083 | File and Directory Discovery | File permissions, sensitive file access |
| T1134 | Access Token Manipulation | Sudo configurations, setuid binaries |
| T1574 | Hijack Execution Flow | PATH hijacking, library injection |
| T1053 | Scheduled Task/Job | Cron job security, timer configurations |
| T1552 | Unsecured Credentials | World-readable files, history files |
| T1040 | Network Sniffing | Network configuration, firewall rules |
| T1027 | Obfuscated Files | File permissions audit |
| T1613 | Container and Resource Discovery | Docker, LXC configuration |
| T1190 | Exploit Public-Facing Application | Web server configuration |
| T1059 | Command and Scripting Interpreter | Shell configuration, script permissions |
| T1070 | Indicator Removal | Log configuration, audit settings |
| T1562 | Impair Defenses | Firewall, SELinux, AppArmor status |

## Resources

- [GitHub Repository](https://github.com/CISOfy/lynis)
- [Lynis Documentation](https://cisofy.com/documentation/lynis/)
- [Kali Linux Tool Page](https://www.kali.org/tools/lynis/)
- [Linux Audit Blog](https://linux-audit.com/)
- [CISOfy Software Packages](https://packages.cisofy.com/)
- [Lynis Enterprise](https://cisofy.com/lynis-enterprise/)
- [HackTricks - Linux Privilege Escalation](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html)
- [Lynis SDK (Custom Tests)](https://github.com/CISOfy/lynis-sdk)
- [CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/96)
- [Lynis Man Page](https://cisofy.com/support/manpage/lynis/)
