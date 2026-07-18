---
tool_name: bed
display_name: BED - Brute Force Exploit Detector
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: fuzzing
install_command: "sudo apt install bed"
version: "0.5"
github_repo: "https://gitlab.com/kalilinux/packages/bed"
kali_tools_page: "https://www.kali.org/tools/bed/"
difficulty_level: beginner
requires_root: false
gui_available: false
tags: [fuzzing, brute-force, protocol-testing, buffer-overflow, format-string]
related_tools: [sfuzz, spike, wfuzz, nikto]
---

# BED - Brute Force Exploit Detector

## 1. Introduction & Overview

BED (Brute Force Exploit Detector) is a simple, scriptable network protocol fuzzer that tests for buffer overflows, format string vulnerabilities, and integer overflows in network services. Unlike coverage-guided fuzzers, BED uses a brute-force approach: it systematically modifies protocol fields with oversized values, format strings, and boundary conditions, watching for crashes.

**Created:** 2006 by Jan Hoofweg (wiregh0)
**License:** GPL v2

### What It Does

BED sends a series of mutated protocol commands to a target service and monitors for:
- **Crashes** (connection drops, core dumps)
- **Unexpected responses** (error messages revealing internal state)
- **Differences** between normal and malformed input responses

It supports built-in protocols: FTP, HTTP, SMTP, IMAP, POP3, SSH, Telnet, and more. Custom protocols can be defined using BED's configuration files.

### When to Use

- **Quick protocol fuzzing**: Test common services with minimal setup
- **Network service audits**: Check if known protocols handle malformed input gracefully
- **CTF challenges**: Find vulnerabilities in custom services by writing protocol profiles
- **Initial vulnerability screening**: Fast first-pass before deploying sophisticated fuzzers

### When NOT to Use

- You need coverage-guided feedback (use AFL instead)
- Target is a file format parser (not network-facing)
- You need to fuzz complex state machines (BED doesn't maintain connection state)
- Deep, long-running fuzzing sessions (BED is better for quick scans)

### Legal & Ethical

BED is a vulnerability research tool. Use it only on systems you own or have explicit authorization to test. Brute-force fuzzing may cause service crashes and data corruption.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Protocol profile** | Configuration file defining valid commands and fields for a service |
| **Brute-force mutation** | Systematic replacement of fields with oversized/invalid values |
| **Baseline comparison** | Compares responses between valid and malformed inputs |
| **Multiple tries** | Sends each mutation multiple times to catch intermittent crashes |

## 2. Installation & Setup

### System Requirements

- Python 3.6+
- Network access to the target
- No special hardware required

### Installation

```bash
sudo apt install bed
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  bed
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 48.3 kB of archives.
After this operation, 196 kB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 bed all 0.5-1kali1 [48.3 kB]
Fetched 48.3 kB in 0s (242 kB/s)
Selecting previously unselected package bed.
Preparing to unpack .../bed_0.5-1kali1_all.deb ...
Unpacking bed (0.5-1kali1) ...
Setting up bed (0.5-1kali1) ...
```

### Configuration

BED profiles are stored in `/usr/share/bed/`. Each profile defines:
- Connection method (TCP/UDP)
- Protocol commands and fields
- Port numbers

## 3. Basic Usage

### First Run

```bash
bed -s FTP -t 192.168.1.50 -p 21 -v 3
```

**Explanation:** Tests the FTP service on 192.168.1.50:21 with verbose output at level 3.

**Example output:**
```
BED: Starting bruite force checks:
  FTP:
    Login:    USER test
    Login:    PASS test
    Cmds:     TEST, CWD, DELE, MKD, RMD, RNFR, RNTO, STOR, RETR, LIST, NLST, APPE, ...
```

### Testing HTTP

```bash
bed -s HTTP -t 192.168.1.50 -p 8080
```

**Explanation:** Fuzzes HTTP methods, headers, and URI parameters on the target web server.

### Testing SSH

```bash
bed -s SSH -t 192.168.1.50 -p 22 -v 2
```

**Explanation:** Sends malformed SSH protocol messages to detect parsing vulnerabilities.

### Flags Reference

| Flag | Description |
|------|-------------|
| `-s SERVICE` | Service/protocol to test (FTP, HTTP, SMTP, etc.) |
| `-t TARGET` | Target IP address or hostname |
| `-p PORT` | Target port number |
| `-v LEVEL` | Verbosity level (0-4) |
| `-d` | Enable debug mode |
| `-o FILE` | Output file for results |
| `-m NUM` | Number of tries per mutation (default: 3) |

### Available Protocols

| Protocol | Default Port | Description |
|----------|-------------|-------------|
| FTP | 21 | File Transfer Protocol |
| HTTP | 80 | Hypertext Transfer Protocol |
| SMTP | 25 | Simple Mail Transfer Protocol |
| IMAP | 143 | Internet Message Access Protocol |
| POP3 | 110 | Post Office Protocol v3 |
| SSH | 22 | Secure Shell |
| TELNET | 23 | Telnet |
| IRC | 6667 | Internet Relay Chat |
| NNTP | 119 | Network News Transfer Protocol |

## 4. Intermediate Usage

### Custom Port for Non-Standard Services

```bash
bed -s HTTP -t 192.168.1.50 -p 8443
```

**Explanation:** Test an HTTP service running on a non-standard port.

### Batch Testing Multiple Hosts

```bash
#!/bin/bash
for host in $(cat targets.txt); do
    echo "=== Testing $host ==="
    bed -s HTTP -t $host -p 80 -o "bed_${host}.txt"
    bed -s FTP -t $host -p 21 -o "bed_ftp_${host}.txt"
done
```

**Explanation:** Automates BED across multiple targets.

### Increasing Reliability

```bash
bed -s FTP -t 192.168.1.50 -p 21 -m 5
```

**Explanation:** Sends each mutation 5 times instead of the default 3, catching intermittent crashes.

### Combining with Nmap

```bash
nmap -sV -p 21,25,80,110,143,993 192.168.1.50
bed -s FTP -t 192.168.1.50 -p 21
bed -s HTTP -t 192.168.1.50 -p 80
```

**Explanation:** Use Nmap to discover running services, then fuzz each one with BED.

## 5. Advanced Usage

### Writing Custom BED Profiles

BED uses XML-based protocol profiles. Create a custom profile for a non-standard service:

```xml
<!-- /usr/share/bed/custom_service.xml -->
<protocol name="custom">
  <connection type="tcp"/>
  <command name="LOGIN">
    <field type="string" value="USER " max_len="1000"/>
    <field type="string" value="\r\n"/>
  </command>
  <command name="QUERY">
    <field type="string" value="QUERY " max_len="1000"/>
    <field type="string" value="\r\n"/>
  </command>
</protocol>
```

### Integration in Penetration Testing Workflow

```bash
# Phase 1: Service discovery
nmap -sV -oX scan.xml 192.168.1.0/24

# Phase 2: Quick BED scan on discovered services
for service in ftp http smtp; do
    port=$(nmap -p - --open 192.168.1.0/24 | grep $service | awk '{print $1}')
    bed -s $service -t 192.168.1.x -p $port -o "bed_${service}.txt"
done

# Phase 3: Deep fuzz with AFL on interesting targets
```

### Monitoring for Subtle Vulnerabilities

```bash
bed -s HTTP -t 192.168.1.50 -p 8080 -v 4
```

**Explanation:** Level 4 verbosity shows all sent/received data, useful for finding information disclosure.

## 6. Real-World Scenarios

### Scenario 1: Router Admin Panel Assessment

**Target:** Router web interface at 192.168.1.1:80

```bash
bed -s HTTP -t 192.168.1.1 -p 80 -v 3
# Look for crash responses or unusual error messages
```

### Scenario 2: VoIP System Testing

**Target:** SIP server on port 5060

```bash
# Write custom SIP profile
bed -s SIP -t 192.168.1.100 -p 5060
```

### Scenario 3: CTF Service Enumeration

**Target:** Custom service on port 9999

```bash
# Quick test with telnet first
echo "HELP" | nc 192.168.1.50 9999
# Then write BED profile based on discovered commands
bed -s CUSTOM -t 192.168.1.50 -p 9999
```

### Scenario 4: Pre-Engagement Screening

**Context:** Quick vulnerability check before deep penetration test

```bash
for host in $(nmap -sn 192.168.1.0/24 | grep "report" | awk '{print $NF}'); do
    bed -s HTTP -t $host -p 80 -o "preengagement_${host}.txt"
done
```

## 7. Detection & Defense

### How Defenders Detect

- **Connection flood**: BED makes rapid, repeated connections with malformed data
- **Error rate spike**: Services log abnormal numbers of parse errors
- **IDS signatures**: Malformed protocol messages trigger protocol anomaly detectors
- **Honeypots**: Some honeypots detect and log BED-like fuzzing patterns

### Mitigation

- Use protocol-aware firewalls that validate input format
- Deploy IDS/IPS with protocol anomaly detection
- Implement connection rate limiting per source IP
- Use fuzzing-resistant libraries (safe parsers, input validation)

### Blue Team Response

```bash
# Monitor connection attempts
netstat -an | grep :8080 | awk '{print $5}' | sort | uniq -c | sort -rn | head -20
# Check for malformed HTTP headers in logs
grep -E "^(GET|POST|PUT|DELETE).{1000,}" /var/log/apache2/access.log
```

## 8. Troubleshooting

### Common Errors

**Service doesn't crash:**
BED tests known vulnerability patterns. If the service handles malformed input gracefully, BED won't find issues — use AFL for deeper fuzzing.

**False positives:**
The service might log errors but remain functional. Check if the service actually crashed by reconnecting.

**Connection refused:**
Verify the target is running and the port is open: `nmap -p PORT TARGET`

### Performance Optimization

- Run BED against one service at a time
- Use `-m 3` (default) for reliable results without excessive noise
- Combine with `tmux` for parallel testing across multiple targets

### FAQ

**Q: How does BED differ from AFL?**
A: BED is a quick, protocol-aware brute-force scanner. AFL is a deep, coverage-guided fuzzer. Use BED for initial screening, AFL for deep analysis.

**Q: Can I fuzz custom protocols?**
A: Yes, by writing custom BED profile files in the protocol definition format.

**Q: Does BED find zero-days?**
A: BED finds known vulnerability patterns. For novel bugs, use AFL or write custom fuzzers.

---

## Resources

- **Kali Tools Page**: https://www.kali.org/tools/bed/
- **BED Documentation**: `/usr/share/bed/` directory on Kali
- **Wiregh0 GitHub**: https://github.com/wiregh0
- **Protocol Fuzzing Guide**: https://resources.infosecinstitute.com/topic/protocol-fuzzing/
