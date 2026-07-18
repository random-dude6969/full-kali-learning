---
title: "protos-sip Cheatsheet"
description: Quick reference for protos-sip SIP protocol test suite
tool: protos-sip
category: voip
difficulty: advanced
---

# protos-sip Cheatsheet

## Quick Start

```bash
# Full test suite
sudo protos-sip -host 192.168.1.100 -verbose

# Category-specific
sudo protos-sip -host 192.168.1.100 -category "header-syntax"

# Single test case
sudo protos-sip -host 192.168.1.100 -testcase TC-001
```

## SIP Methods

| Method | Purpose |
|--------|---------|
| INVITE | Initiate call |
| REGISTER | Register endpoint |
| OPTIONS | Query capabilities |
| BYE | Terminate call |
| CANCEL | Cancel pending |
| ACK | Acknowledge |

## Common Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-host` | Target host | `-host 192.168.1.100` |
| `-port` | SIP port | `-port 5060` |
| `-timeout` | Response timeout | `-timeout 5000` |
| `-verbose` | Detailed output | `-verbose` |
| `-testcase` | Specific test | `-testcase TC-001` |
| `-category` | Test category | `-category header-syntax` |
| `-output` | Save results | `-output results.txt` |
| `-rate` | Send rate | `-rate 10` |

## Examples

### Full Test Suite
```bash
sudo protos-sip -host 192.168.1.100 -verbose -output full_test.txt
```

### Header Syntax Tests
```bash
sudo protos-sip -host 192.168.1.100 -category "header-syntax" -verbose
```

### Transaction Layer Tests
```bash
sudo protos-sip -host 192.168.1.100 -category "transaction-layer" -verbose
```

### Rate-Limited
```bash
sudo protos-sip -host 192.168.1.100 -rate 5 -timeout 10000
```

## Chaining

```bash
# Find SIP hosts, then test
nmap -sU -p 5060 192.168.1.0/24 | grep "open" | awk '{print $2}' | while read ip; do
    sudo protos-sip -host $ip -category "header-syntax" -output "protos_${ip}.txt"
done
```

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Permission denied | Run with `sudo` |
| No test cases | Reinstall package |
| Connection timeout | Increase `-timeout` |
| Tool hangs | Reduce `-rate` |

```bash
# Verify connectivity
nc -zvu 192.168.1.100 5060

# Check test cases
ls /usr/local/share/protos-sip/testcases/
```
