# DefectDojo - Vulnerability Management Platform

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

DefectDojo is an open-source application security vulnerability management platform that helps organizations manage their application security program. It provides centralized vulnerability tracking, management, and reporting across multiple security tools and assessments.

### 1.1 What is DefectDojo?

DefectDojo is a security orchestration and vulnerability management platform that:
- Aggregates findings from multiple security tools
- Manages vulnerabilities through their lifecycle
- Provides metrics and dashboards
- Integrates with CI/CD pipelines
- Generates comprehensive reports

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Type | Vulnerability Management Platform |
| Version | 2.37.3 |
| License | BSD |
| Language | Python (Django) |
| Database | PostgreSQL |
| Port | 8080 (Web) |
| GitHub | DefectDojo/django-DefectDojo |

### 1.3 DefectDojo Features

| Feature | Description |
|---------|-------------|
| Tool Integration | Import from 20+ security tools |
| Vulnerability Management | Track through lifecycle |
| Metrics & Dashboards | Real-time visibility |
| CI/CD Integration | Automate security testing |
| Report Generation | Customizable reports |
| API Access | RESTful API |
| Multi-tenancy | Support multiple teams |

### 1.4 Why Use DefectDojo?

- **Centralized Management:** Single source of truth for vulnerabilities
- **Tool Aggregation:** Combine results from multiple scanners
- **Metrics:** Track security posture over time
- **Automation:** Integrate with development workflows
- **Compliance:** Meet regulatory requirements
- **Collaboration:** Enable team coordination

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install defectdojo

# Start the service
sudo defectdojo-start

# Access at http://127.0.0.1:8080

# Stop the service
sudo defectdojo-stop
```

### 2.2 Docker

```bash
# Clone repository
git clone https://github.com/DefectDojo/django-DefectDojo.git
cd django-DefectDojo

# Start with Docker Compose
docker compose up -d

# Access at http://localhost:8080
```

### 2.3 Docker Compose Configuration

```yaml
version: '3.8'
services:
  nginx:
    image: defectdojo/defectdojo-nginx:latest
    ports:
      - "8080:8080"
    depends_on:
      - uwsgi
    volumes:
      - ./docker/nginx.conf:/etc/nginx/conf.d/defectdojo.conf

  uwsgi:
    image: defectdojo/defectdojo-uwsgi:latest
    environment:
      - DD_DATABASE_HOST=database
      - DD_DATABASE_PORT=5432
      - DD_DATABASE_NAME=defectdojo
      - DD_DATABASE_USER=defectdojo
      - DD_DATABASE_PASSWORD=defectdojo
    volumes:
      - ./docker/uwsgi.ini:/opt/defectdojo/uwsgi.ini

  celery:
    image: defectdojo/defectdojo-celery:latest
    environment:
      - DD_DATABASE_HOST=database
      - DD_DATABASE_PORT=5432
      - DD_DATABASE_NAME=defectdojo
      - DD_DATABASE_USER=defectdojo
      - DD_DATABASE_PASSWORD=defectdojo

  database:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=defectdojo
      - POSTGRES_USER=defectdojo
      - POSTGRES_PASSWORD=defectdojo
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 2.4 First-Time Setup

1. Navigate to `http://localhost:8080`
2. Create admin account
3. Configure integrations
4. Import first scan results

---

## 3. Interface Overview

### 3.1 Dashboard

```
+------------------------------------------------------------------+
|  DefectDojo  |  Dashboard  |  Products  |  Engagements  |  API  |
+------------------------------------------------------------------+
|            |                                                    |
|  Metrics   |  Dashboard View                                    |
|  - Critical|                                                    |
|  - High    |  Vulnerability counts, trends, metrics            |
|  - Medium  |                                                    |
|  - Low     |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Recent Activity  |  Tool Integrations  |  Reports             |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **Dashboard:** Overview of security posture
- **Products:** Applications and systems
- **Engagements:** Testing activities
- **Findings:** Individual vulnerabilities
- **Reports:** Generated documentation
- **Settings:** Configuration options

---

## 4. Product and Engagement Management

### 4.1 Creating Products

```
Products > Add Product > Fill details > Save
```

Product fields:
- **Name:** Application/system name
- **Description:** Brief description
- **Product Type:** Web, Mobile, Network, etc.
- **Business Criticality:** Critical, High, Medium, Low

### 4.2 Creating Engagements

```
Engagements > Add Engagement > Select product > Configure > Save
```

Engagement types:
- **Interactive:** Manual testing
- **CI/CD:** Automated scanning
- **Weekly:** Regular scans
- **One-time:** Ad-hoc testing

---

## 5. Importing Findings

### 5.1 Supported Tools

| Tool | Import Format |
|------|---------------|
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

### 5.2 Importing via UI

```
Engagements > Select engagement > Import Scan Results > Upload file
```

### 5.3 Importing via API

```bash
# Import scan results
curl -X POST http://localhost:8080/api/v2/import-scan/ \
    -H "Authorization: Token YOUR_TOKEN" \
    -F "file=@scan_results.xml" \
    -F "scan_type=Burp Suite" \
    -F "product_name=MyApp" \
    -F "engagement_name=Weekly Scan"
```

---

## 6. Vulnerability Management Lifecycle

### 6.1 Finding States

| State | Description |
|-------|-------------|
| Open | New finding |
| Verified | Confirmed as valid |
| False Positive | Not a real vulnerability |
| Mitigated | Fix applied |
| Closed | Verified fixed |
| Inherited | From parent product |

### 6.2 Managing Findings

```
Findings > Select finding > Update status > Add notes
```

### 6.3 Finding Fields

| Field | Description |
|-------|-------------|
| Title | Finding name |
| Severity | Critical/High/Medium/Low/Info |
| CVSS Score | Vulnerability score |
| Description | Detailed explanation |
| Mitigation | Remediation steps |
| References | CVE links, etc. |
| Impact | Business impact |

---

## 7. Metrics and Dashboards

### 7.1 Dashboard Metrics

- **Total Findings:** All vulnerabilities
- **Open Findings:** Active issues
- **Critical/High:** High-priority items
- **Mean Time to Close:** Resolution time
- **Finding Distribution:** By severity

### 7.2 Custom Reports

```
Reports > Generate Report > Select metrics > Configure > Generate
```

### 7.3 Metrics API

```bash
# Get metrics
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/metrics/
```

---

## 8. CI/CD Integration

### 8.1 GitHub Actions

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run security scan
        run: bandit -r . -f json -o bandit.json
      - name: Upload to DefectDojo
        run: |
          curl -X POST http://defectdojo.example.com/api/v2/import-scan/ \
            -H "Authorization: Token ${{ secrets.DEFECTDOJO_TOKEN }}" \
            -F "file=@bandit.json" \
            -F "scan_type=Bandit" \
            -F "product_name=${{ github.event.repository.name }}" \
            -F "engagement_name=CI/CD Scan"
```

### 8.2 Jenkins Integration

```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Security Scan') {
            steps {
                sh 'bandit -r . -f json -o bandit.json'
                sh '''
                    curl -X POST http://defectdojo/api/v2/import-scan/ \
                        -H "Authorization: Token ${DEFECTDOJO_TOKEN}" \
                        -F "file=@bandit.json" \
                        -F "scan_type=Bandit" \
                        -F "product_name=${JOB_NAME}" \
                        -F "engagement_name=Jenkins Build"
                '''
            }
        }
    }
}
```

---

## 9. API Usage

### 9.1 Authentication

```bash
# Get API token
# UI > Settings > API Tokens > Generate

# Use token in requests
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/
```

### 9.2 Common API Endpoints

```bash
# List products
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/products/

# List findings
curl -H "Authorization: Token YOUR_TOKEN" \
    http://localhost:8080/api/v2/findings/

# Create finding
curl -X POST http://localhost:8080/api/v2/findings/ \
    -H "Authorization: Token YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"title":"Test Finding","severity":"High"}'
```

---

## 10. Best Practices

### 10.1 Workflow

1. **Setup:** Create products and engagements
2. **Import:** Bring in scan results regularly
3. **Triage:** Review and categorize findings
4. **Assign:** Distribute work to team
5. **Remediate:** Fix vulnerabilities
6. **Verify:** Confirm fixes
7. **Report:** Generate metrics and reports

### 10.2 Organization

- Use consistent naming conventions
- Tag findings for filtering
- Create templates for common findings
- Document remediation steps

### 10.3 Integration

- Automate scan imports
- Set up notifications
- Integrate with ticketing systems
- Regular metric reviews

---

## 11. Troubleshooting

### Common Issues

**Problem:** Service won't start

```bash
# Check status
sudo systemctl status defectdojo

# Check logs
tail -f /var/log/defectdojo/production.log

# Restart service
sudo systemctl restart defectdojo
```

**Problem:** Import errors

```bash
# Check file format
# Verify scan type
# Check API token
# Review logs
```

**Problem:** Database issues

```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Run migrations
defectdojo-manage migrate
```

---

## 12. Advanced Features

### 12.1 Finding Templates

Create templates for common vulnerability types:

```
Findings > Templates > Add Template
```

Template fields:
- **Title Pattern:** Naming convention
- **Severity Default:** Starting severity
- **Description Template:** Standard description
- **Mitigation Template:** Standard fix

### 12.2 Custom Fields

Add custom fields to findings:

```
System > Custom Fields > Add Field
```

Custom field types:
- **Text:** Free text input
- **Number:** Numeric values
- **Boolean:** True/False
- **Date:** Date values
- **Select:** Dropdown options

### 12.3 SLA Management

Configure Service Level Agreements:

```
System > SLA > Configure
```

SLA settings:
- **Critical:** 24 hours
- **High:** 7 days
- **Medium:** 30 days
- **Low:** 90 days

### 12.4 Notifications

Configure email notifications:

```
System > Notifications > Configure
```

Notification events:
- New finding created
- Finding status changed
- SLA deadline approaching
- Report generated

---

## 13. Compliance and Standards

### 13.1 Supported Standards

| Standard | Description |
|----------|-------------|
| OWASP Top Ten | Web application security |
| SANS Top 25 | Most dangerous software flaws |
| NIST CSF | Cybersecurity Framework |
| PCI DSS | Payment card security |
| HIPAA | Healthcare data protection |

### 13.2 Mapping Findings to Standards

```
Findings > Select Finding > Compliance > Map to Standard
```

### 13.3 Compliance Reporting

```
Reports > Compliance > Select Standard > Generate
```

---

## 14. Data Export and Integration

### 14.1 Export Options

| Format | Use Case |
|--------|----------|
| PDF | Formal reports |
| CSV | Data analysis |
| JSON | API integration |
| XML | Tool integration |
| Excel | Spreadsheet analysis |

### 14.2 Webhook Integration

```bash
# Configure webhooks
# System > Webhooks > Add Webhook

# Webhook events
- finding.created
- finding.updated
- engagement.created
```

### 14.3 JIRA Integration

```
System > Integrations > JIRA > Configure
```

JIRA settings:
- **Server URL:** JIRA instance
- **API Token:** Authentication
- **Project Key:** Target project
- **Issue Type:** Finding mapping

---

## 15. Performance Optimization

### 15.1 Database Optimization

```bash
# Optimize PostgreSQL
vacuumdb -U defectdojo defectdojo

# Analyze tables
analyzedb -U defectdojo defectdojo

# Reindex
reindexdb -U defectdojo defectdojo
```

### 15.2 Cache Configuration

```bash
# Enable Redis cache
# System > Settings > Cache > Redis

# Configure cache timeout
export DEFECTDOJO_CACHE_TIMEOUT=300
```

### 15.3 Celery Workers

```bash
# Check Celery status
celery -A dojo inspect active

# Scale workers
celery -A dojo worker --concurrency=4
```

---

## 16. Backup and Recovery

### 16.1 Database Backup

```bash
# Backup PostgreSQL
pg_dump -U defectdojo defectdojo > backup.sql

# Schedule daily backups
0 2 * * * pg_dump -U defectdojo defectdojo > /backup/defectdojo_$(date +\%Y\%m\%d).sql
```

### 16.2 File Backup

```bash
# Backup media files
tar -czf media_backup.tar.gz /var/lib/defectdojo/media/

# Backup configuration
tar -czf config_backup.tar.gz /etc/defectdojo/
```

### 16.3 Recovery Procedures

```bash
# Restore database
psql -U defectdojo defectdojo < backup.sql

# Restore media
tar -xzf media_backup.tar.gz -C /

# Restart services
sudo systemctl restart defectdojo
```

---

## References

- [DefectDojo Official Website](https://www.defectdojo.org/)
- [DefectDojo GitHub](https://github.com/DefectDojo/django-DefectDojo)
- [DefectDojo Documentation](https://defectdojo.github.io/django-DefectDojo/)
- [Kali Linux DefectDojo Package](https://www.kali.org/tools/defectdojo/)
- [DefectDojo API Documentation](https://defectdojo.github.io/django-DefectDojo/integrations/api-v2/)
