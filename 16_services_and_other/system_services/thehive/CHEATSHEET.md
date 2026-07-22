# TheHive - Quick Reference Cheatsheet

---

## Quick Start

```bash
# Docker installation
docker pull strangebee/thehive:latest
docker run -d --name thehive -p 9000:9000 strangebee/thehive:latest

# Access at http://localhost:9000
```

---

## Default Configuration

| Setting | Value |
|---------|-------|
| Web UI | `http://localhost:9000` |
| Port | 9000 |
| Database | Elasticsearch/Cassandra |
| Language | Scala |
| License | AGPL v3 |

---

## Key Features

| Feature | Description |
|---------|-------------|
| Case Management | Track incidents through lifecycle |
| Alert Analysis | Process alerts from multiple sources |
| Task Management | Assign and track response tasks |
| Observable Analysis | Analyze IoCs and artifacts |
| MISP Integration | Threat intelligence sharing |
| Cortex Integration | Automated analysis |
| REST API | Automation and integration |

---

## Docker Compose Setup

```yaml
version: '3.8'
services:
  thehive:
    image: strangebee/thehive:latest
    ports:
      - "9000:9000"
    depends_on:
      - elasticsearch
      - cassandra

  cortex:
    image: strangebee/cortex:latest
    ports:
      - "9001:9001"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.0
    environment:
      - discovery.type=single-node

  cassandra:
    image: cassandra:4.0
```

---

## Case Lifecycle

```
New → In Progress → Resolved → Closed
```

### Severity Levels
| Level | Description |
|-------|-------------|
| Critical | Immediate response required |
| High | Urgent attention needed |
| Medium | Standard response |
| Low | Monitor and assess |

---

## Alert Workflow

```
New → Under Review → Ignored | Created Case
```

---

## Observable Types

| Type | Example |
|------|---------|
| IP Address | 192.168.1.1 |
| Domain | evil.com |
| URL | http://evil.com/malware |
| Email | attacker@evil.com |
| File | malware.exe |
| Hash | MD5/SHA1/SHA256 |

---

## API Usage

```bash
# Get API key: UI > Administration > Users > API Key

# List cases
curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:9000/api/v1/case

# Create case
curl -X POST http://localhost:9000/api/v1/case \
    -H "Authorization: Bearer YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"title":"Incident","severity":"High"}'

# List alerts
curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:9000/api/v1/alert

# Add observable
curl -X POST http://localhost:9000/api/v1/case/{caseId}/observable \
    -H "Authorization: Bearer YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"data":"192.168.1.1","dataType":"ip"}'
```

---

## Incident Response Workflow

```
1. Detection    - Receive alert
2. Triage       - Assess severity
3. Investigation - Create case, add observables
4. Containment  - Isolate affected systems
5. Eradication  - Remove threat
6. Recovery     - Restore systems
7. Lessons Learned - Document and improve
```

---

## Cortex Analyzers

| Analyzer | Purpose |
|----------|---------|
| VirusTotal | Malware analysis |
| Shodan | IP reconnaissance |
| AbuseIPDB | IP reputation |
| MaxMind | GeoIP lookup |
| YARA | Malware detection |

---

## MISP Integration

```bash
# Configure MISP
# Administration > MISP > Add Server

# Import events
# Cases > Import from MISP

# Export observables
# Case > Observables > Export to MISP
```

---

## Troubleshooting

```bash
# Check TheHive logs
docker logs thehive

# Check Elasticsearch
curl http://localhost:9200

# Check Cassandra
nodetool status

# Restart services
docker restart thehive
docker restart cortex
docker restart elasticsearch
docker restart cassandra
```

---

## TLP (Traffic Light Protocol)

| TLP | Color | Description |
|-----|-------|-------------|
| TLP:RED | Red | Restricted distribution |
| TLP:AMBER+STRICT | Amber | Limited distribution |
| TLP:AMBER | Amber | Organization-wide |
| TLP:GREEN | Green | Community-wide |
| TLP:CLEAR | Clear | Unlimited distribution |

---

## Useful Commands

```bash
# Check TheHive status
curl http://localhost:9000/api/v1/status

# Check Cortex status
curl http://localhost:9001/api/status

# View Docker logs
docker logs -f thehive

# Backup data
docker exec thehive tar -czf /opt/thp/thehive/data/backup.tar.gz /opt/thp/thehive/data
```
