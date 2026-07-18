---
title: "mxcheck - Email Server and MX Record Testing"
description: "Email server testing tool that verifies MX records, tests SMTP connectivity, and checks SPF/DKIM/DMARC configurations"
mitre_tactic: "discovery"
mitre_tactic_id: "TA0007"
mitre_subcategory: "mxcheck"
categories:
  - "email"
  - "dns"
  - "smtp"
  - "security"
tags:
  - "email"
  - "mx"
  - "smtp"
  - "spf"
  - "dkim"
  - "dmarc"
install_command: "sudo apt install mxcheck"
version: "0.5"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/mxcheck/"
difficulty_level: "beginner"
requires_root: false
gui_available: false
related_tools:
  - "swaks"
  - "smtp-user-enum"
  - "dig"
  - "telnet"
---

# mxcheck - Email Server and MX Record Testing

## 1. Introduction

**mxcheck** is an email server testing tool that verifies MX records, tests SMTP connectivity, and checks email security configurations including SPF, DKIM, and DMARC. The tool provides comprehensive email infrastructure analysis for security assessments and troubleshooting.

### Key Capabilities

**MX Record Verification**: Validates mail exchanger records for domains.

**SMTP Testing**: Tests SMTP server connectivity and functionality.

**SPF Checking**: Verifies Sender Policy Framework configuration.

**DKIM Validation**: Checks DomainKeys Identified Mail setup.

**DMARC Analysis**: Examines Domain-based Message Authentication, Reporting, and Conformance.

### Use Cases

- **Email Security Auditing**: Verify email security configurations.
- **Penetration Testing**: Identify email infrastructure vulnerabilities.
- **Troubleshooting**: Diagnose email delivery issues.
- **Compliance Verification**: Ensure email configurations meet standards.
- **Incident Response**: Investigate email-related security incidents.

### How mxcheck Works

mxcheck queries DNS for MX records, connects to identified mail servers, and performs various tests to validate email configurations. It checks SMTP responses, verifies email authentication mechanisms, and reports on potential security issues.

## 2. Installation

### Standard Installation

```bash
sudo apt update
sudo apt install mxcheck
```

### Verify Installation

```bash
mxcheck --version
```

### Dependencies

mxcheck requires DNS resolution capabilities and network access for SMTP testing.

## 3. Core Concepts

### MX Records

**Mail Exchanger (MX)**: DNS record specifying mail servers for a domain.

**Priority**: Lower numbers indicate higher priority mail servers.

**Multiple MX Records**: Domains can have multiple mail servers for redundancy.

### SMTP Protocol

**Simple Mail Transfer Protocol**: Standard protocol for sending email.

**SMTP Commands**: HELO, MAIL FROM, RCPT TO, DATA, QUIT.

**SMTP Responses**: 2xx success, 4xx temporary failure, 5xx permanent failure.

### Email Authentication

**SPF (Sender Policy Framework)**: Specifies authorized sending servers for a domain.

**DKIM (DomainKeys Identified Mail)**: Cryptographic signature verifying email origin.

**DMARC (Domain-based Message Authentication, Reporting, and Conformance)**: Policy framework for SPF and DKIM alignment.

### Security Considerations

**Open Relay**: Mail server that accepts and forwards email from unauthorized sources.

**Email Spoofing**: forging email headers to impersonate senders.

**Phishing**: fraudulent emails designed to steal credentials or information.

## 4. Command Reference

### Basic Syntax

```bash
mxcheck [OPTIONS] DOMAIN
```

### Core Flags

**-p PORT**: Specify SMTP port (default: 25).

**-s**: Perform SMTP test.

**-v**: Verbose output. Show detailed results.

**-t TIMEOUT**: Set connection timeout in seconds.

**-d**: Debug mode. Show DNS query details.

### Email Testing Flags

**-f EMAIL**: Set FROM email address for SMTP test.

**-r EMAIL**: Set RCPT TO email address for SMTP test.

**-h HOSTNAME**: Set HELO/EHLO hostname.

### Output Flags

**-o FILE**: Write results to file.

**-q**: Quiet mode. Minimal output.

**--no-colour**: Disable colored output.

## 5. Examples

### Basic MX Record Check

```bash
mxcheck example.com
```

**Result**: Queries and displays MX records for domain.

### SMTP Test

```bash
mxcheck -s example.com
```

**Result**: Tests SMTP connectivity to mail servers.

### Verbose Output

```bash
mxcheck -v example.com
```

**Result**: Shows detailed MX and SMTP information.

### Custom Port

```bash
mxcheck -p 587 example.com
```

**Result**: Tests submission port instead of default SMTP.

### SMTP with Email Addresses

```bash
mxcheck -s -f sender@example.com -r recipient@example.com example.com
```

**Result**: Tests SMTP with specific email addresses.

### Debug Mode

```bash
mxcheck -d example.com
```

**Result**: Shows DNS query details.

### File Output

```bash
mxcheck -o results.txt example.com
```

**Result**: Saves results to file.

### Extended Timeout

```bash
mxcheck -t 10 example.com
```

**Result**: Uses 10-second timeout for slow servers.

### Multiple Domains

```bash
for domain in example.com test.com demo.com; do
  echo "=== $domain ==="
  mxcheck -v "$domain"
done
```

**Result**: Tests multiple domains sequentially.

### Custom Hostname

```bash
mxcheck -h mail.example.com example.com
```

**Result**: Uses custom HELO hostname.

## 6. Advanced Techniques

### Comprehensive Email Security Audit

```bash
#!/bin/bash
DOMAINS="domains.txt"
OUTPUT_DIR="email_audit_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$OUTPUT_DIR"

while IFS= read -r domain; do
  echo "=== $domain ===" | tee "$OUTPUT_DIR/$domain.txt"
  mxcheck -v "$domain" >> "$OUTPUT_DIR/$domain.txt" 2>&1
  echo "" >> "$OUTPUT_DIR/$domain.txt"
done < "$DOMAINS"

echo "Audit results saved to $OUTPUT_DIR"
```

### SPF/DKIM/DMARC Analysis

```bash
#!/bin/bash
DOMAIN="example.com"

echo "=== MX Records ==="
dig MX "$DOMAIN" +short

echo "=== SPF Record ==="
dig TXT "$DOMAIN" +short | grep -i spf

echo "=== DKIM Record ==="
dig TXT "default._domainkey.$DOMAIN" +short

echo "=== DMARC Record ==="
dig TXT "_dmarc.$DOMAIN" +short

echo "=== SMTP Test ==="
mxcheck -s "$DOMAIN"
```

### Email Infrastructure Mapping

```bash
#!/bin/bash
NETWORK="192.168.1.0/24"

echo "Scanning for email servers..."
nmap -p 25,587,465 "$NETWORK" -oG - | grep "open" | while read line; do
  host=$(echo "$line" | awk '{print $2}')
  echo "=== $host ==="
  mxcheck -s "$host"
done
```

### SMTP Relay Testing

```bash
#!/ bash
# Test for open relay
DOMAIN="target.example.com"
FROM="attacker@evil.com"
RCPT="victim@example.com"

mxcheck -s -f "$FROM" -r "$RCPT" "$DOMAIN"
```

### Email Security Monitoring

```bash
#!/bin/bash
DOMAIN="example.com"
LOGFILE="email_security_$(date +%Y%m%d).log"

echo "$(date): Checking $DOMAIN" >> "$LOGFILE"
mxcheck -v "$DOMAIN" >> "$LOGFILE" 2>&1
echo "---" >> "$LOGFILE"
```

### Comparison Testing

```bash
# Compare email configurations between domains
diff <(mxcheck -v domain1.com) <(mxcheck -v domain2.com)
```

### Batch MX Verification

```bash
#!/bin/bash
# Verify MX records for multiple domains
DOMAINS="domain1.com domain2.com domain3.com"

for domain in $DOMAINS; do
  echo "=== $domain ==="
  MX=$(dig MX "$domain" +short | head -1)
  if [ -n "$MX" ]; then
    echo "MX: $MX"
    mxcheck -s "$domain"
  else
    echo "No MX records found"
  fi
done
```

## 7. Detection & Defense

### Network Detection

**SMTP Scanning**: Detect hosts performing SMTP reconnaissance.

**DNS Query Monitoring**: Monitor for unusual MX record queries.

**Email Testing Patterns**: Identify systematic email infrastructure testing.

### System Detection

**SMTP Logs**: Review mail server logs for testing activity.

**Connection Monitoring**: Track SMTP connection attempts.

### Defense Strategies

**SPF Implementation**: Configure SPF records to authorize sending servers.

**DKIM Setup**: Implement DKIM signing for email authentication.

**DMARC Policy**: Configure DMARC to enforce email authentication.

**SMTP Restrictions**: Limit SMTP access to authorized clients.

**Rate Limiting**: Implement SMTP connection rate limiting.

**Email Authentication**: Require authentication for SMTP submission.

## 8. Troubleshooting

### No MX Records

**DNS Resolution**: Verify domain exists and has MX records.

```bash
dig MX example.com +short
```

**Domain Configuration**: Check domain DNS settings.

### SMTP Connection Failed

**Port Verification**: Confirm SMTP port is open.

```bash
nmap -p 25,587 example.com
```

**Firewall Issues**: Check for firewall blocking SMTP traffic.

**Service Status**: Verify mail server is running.

### Authentication Errors

**Credentials**: Verify SMTP authentication credentials.

**TLS/SSL**: Check for required encryption.

**Permissions**: Ensure account has sending permissions.

### Timeout Issues

**Network Latency**: Increase timeout with -t flag.

```bash
mxcheck -t 10 example.com
```

**Server Load**: Target server may be overloaded.

### SPF/DKIM/DMARC Issues

**Record Verification**: Use dig to verify DNS records.

```bash
dig TXT example.com +short
```

**Syntax Check**: Verify record syntax is correct.

**Propagation**: DNS changes may take time to propagate.

### SMTP Response Errors

**Server Configuration**: Check mail server configuration.

**Relay Settings**: Verify relay authorization.

**Queue Issues**: Check mail queue for stuck messages.

## Resources

- **mxcheck Kali Documentation**: https://www.kali.org/tools/mxcheck/
- **MX Record RFC**: https://tools.ietf.org/html/rfc5321
- **SPF Specification**: https://tools.ietf.org/html/rfc7208
- **DKIM Specification**: https://tools.ietf.org/html/rfc6376
- **DMARC Specification**: https://tools.ietf.org/html/rfc7489

## Related Tools

**swaks**: Swiss Army Knife for SMTP testing with advanced features.

**smtp-user-enum**: SMTP user enumeration tool for identifying valid accounts.

**dig**: DNS lookup tool for querying MX, SPF, DKIM, and DMARC records.

**telnet**: Manual SMTP testing via raw connection.
