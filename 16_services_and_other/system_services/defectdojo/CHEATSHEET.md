# DefectDojo - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Kali Linux
sudo apt install defectdojo
sudo defectdojo-start
# Access at http://127.0.0.1:8080

# Docker
git clone https://github.com/DefectDojo/django-DefectDojo.git
cd django-DefectDojo
docker compose up -d

# Stop service
sudo defectdojo-stop
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Web UI | `http://127.0.0.1:8080` |
| Port | 8080 |
| Version | 2.37.3 |
| Database | PostgreSQL |
| Language | Python (Django) |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Tool Integration | Import from 20+ security tools |
| Vulnerability Management | Track through lifecycle |
| Metrics & Dashboards | Real-time visibility |
| CI/CD Integration | Automate security testing |
| Report Generation | Customizable reports |
| REST API | Automation and integration |
| Multi-tenancy | Support multiple teams |

---

## Supported Import Tools

| Tool | Format |
|------|--------|
| Burp Suite | XML |
| Nessus | XML |
| OpenVAS | XML |
| Qualys | CSV |
| SonarQube | JSON |
| Checkmarx | XML |
| Fortify | XML |
| Snyk | JSON |
| Bandit | JSON |
| Semgrep | JSON |

---

## Finding States

| State | Description |
|-------|-------------|
| Open | New finding |
| Verified | Confirmed valid |
| False Positive | Not real |
| Mitigated | Fix applied |
| Closed | Verified fixed |
| Inherited | From parent |

---

## API Usage

```bash
# Get API token: UI > Settings > API Tokens

# List products
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/products/

# List findings
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/findings/

# Import scan
curl -X POST http://localhost:8080/api/v2/import-scan/ \
    -H "Authorization: Token YOUR_TOKEN" \
    -F "file=@scan.xml" \
    -F "scan_type=Burp Suite" \
    -F "product_name=MyApp" \
    -F "engagement_name=Scan"
```

---

## Workflow

```
1. Create Product
2. Create Engagement
3. Import Scan Results
4. Triage Findings
5. Assign to Team
6. Remediate
7. Verify
8. Report
```

---

## CI/CD Integration

```yaml
# GitHub Actions
- name: Upload to DefectDojo
  run: |
    curl -X POST http://defectdojo/api/v2/import-scan/ \
      -H "Authorization: Token ${{ secrets.DEFECTDOJO_TOKEN }}" \
      -F "file=@scan.json" \
      -F "scan_type=Bandit" \
      -F "product_name=${{ github.event.repository.name }}"
```

---

## Troubleshooting

```bash
# Check service status
sudo systemctl status defectdojo

# View logs
tail -f /var/log/defectdojo/production.log

# Restart service
sudo systemctl restart defectdojo

# Check database
sudo systemctl status postgresql

# Run migrations
defectdojo-manage migrate
```

---

## Report Types

| Report | Description |
|--------|-------------|
| Finding Report | Individual vulnerability |
| Product Report | All findings for product |
| Engagement Report | Testing period results |
| Metrics Report | Dashboard metrics |
| Executive Summary | High-level overview |

---

## Product Types

| Type | Description |
|------|-------------|
| Web Application | Websites |
| Mobile Application | iOS/Android apps |
| Network | Infrastructure |
| Library | Code libraries |
| API | REST/GraphQL APIs |
| Cloud | Cloud services |
