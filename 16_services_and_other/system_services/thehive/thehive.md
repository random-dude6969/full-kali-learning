# TheHive - Security Incident Response Platform

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

TheHive is an open-source Security Incident Response Platform designed to help security teams collaborate during incident response. It provides a centralized platform for managing security incidents, analyzing alerts, and coordinating response activities. TheHive integrates with various security tools and threat intelligence sources to streamline the incident response workflow.

### 1.1 What is TheHive?

TheHive is a scalable, open-source Security Incident Response Platform that:
- Manages security incidents collaboratively
- Analyzes alerts from multiple sources
- Integrates with Cortex for automatic analysis
- Connects to MISP for threat intelligence
- Provides case management capabilities
- Supports customizable workflows

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Type | Security Incident Response Platform |
| License | GNU AGPL v3 |
| Language | Scala |
| Database | Elasticsearch/JanusGraph |
| Port | 9000 (Web) |
| GitHub | TheHive-Project/TheHive |

### 1.3 TheHive Features

| Feature | Description |
|---------|-------------|
| Case Management | Track incidents through lifecycle |
| Alert Analysis | Process alerts from multiple sources |
| Task Management | Assign and track response tasks |
| Observable Analysis | Analyze IoCs and artifacts |
| MISP Integration | Threat intelligence sharing |
| Cortex Integration | Automated analysis |
| Customizable Templates | Standardized workflows |
| REST API | Automation and integration |

### 1.4 TheHive vs Other IR Platforms

| Feature | TheHive | TheHive 4 | Demisto/XSOAR | Splunk SOAR |
|---------|---------|-----------|---------------|-------------|
| License | AGPL | AGPL | Commercial | Commercial |
| Cost | Free | Free | Paid | Paid |
| Scalability | Good | Better | Enterprise | Enterprise |
| Integration | Many | More | Extensive | Extensive |
| Learning Curve | Medium | Medium | High | High |

### 1.5 Why Use TheHive for Incident Response?

- **Centralized Management:** Single platform for all incidents
- **Collaboration:** Team-based incident handling
- **Automation:** Integrate with Cortex for automated analysis
- **Threat Intelligence:** Connect with MISP for IoC sharing
- **Metrics:** Track response times and effectiveness
- **Open Source:** No licensing costs

---

## 2. Architecture

### 2.1 System Components

```
+------------------------------------------------------------------+
|                        TheHive Architecture                      |
+------------------------------------------------------------------+
|                                                                  |
|  +----------------+    +----------------+    +----------------+  |
|  |    TheHive     |    |     Cortex     |    |      MISP      |  |
|  |  (Web UI/API)  |    |  (Analysis)    |    |  (Threat Intel) |  |
|  +----------------+    +----------------+    +----------------+  |
|         |                    |                    |              |
|         +--------------------+--------------------+              |
|                          |                                       |
|                    +-----------+                                 |
|                    |Database  |                                 |
|                    |(ES/Graph)|                                 |
|                    +-----------+                                 |
|                          |                                       |
+------------------------------------------------------------------+
```

### 2.2 Component Descriptions

| Component | Purpose |
|-----------|---------|
| TheHive | Main application, web interface, API |
| Cortex | Automated analysis and response |
| MISP | Threat intelligence platform |
| Elasticsearch | Data storage and search |
| JanusGraph | Graph database for relationships |

---

## 3. Installation

### 3.1 Kali Linux

```bash
# TheHive is not pre-installed on Kali
# Install from repository or Docker

# Docker installation
docker pull strangebee/thehive:latest

# Run TheHive
docker run -d --name thehive \
    -p 9000:9000 \
    -v thehive-data:/opt/thp/thehive/data \
    strangebee/thehive:latest
```

### 3.2 Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  thehive:
    image: strangebee/thehive:latest
    ports:
      - "9000:9000"
    environment:
      - THEHIVE_ENDPOINT=http://localhost:9000
    volumes:
      - thehive-data:/opt/thp/thehive/data
    depends_on:
      - elasticsearch
      - cassandra

  cortex:
    image: strangebee/cortex:latest
    ports:
      - "9001:9001"
    volumes:
      - cortex-data:/opt/Cortex/data

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - es-data:/usr/share/elasticsearch/data

  cassandra:
    image: cassandra:4.0
    volumes:
      - cassandra-data:/var/lib/cassandra/data

volumes:
  thehive-data:
  cortex-data:
  es-data:
  cassandra-data:
```

### 3.3 From Source

```bash
# Install dependencies
sudo apt install elasticsearch cassandra

# Download TheHive
wget https://github.com/TheHive-Project/TheHive/releases/latest

# Extract and install
tar -xzf TheHive-*.tar.gz
cd TheHive-*

# Configure
cp conf/application.conf.sample conf/application.conf

# Start
./bin/thehive
```

### 3.4 First-Time Setup

1. Navigate to `http://localhost:9000`
2. Create admin account
3. Configure organization
4. Set up integrations
5. Create first case

---

## 4. Interface Overview

### 4.1 Dashboard

```
+------------------------------------------------------------------+
|  TheHive  |  Cases  |  Alerts  |  Observables  |  Dashboards     |
+------------------------------------------------------------------+
|            |                                                    |
|  Sidebar   |              Main Content Area                    |
|            |                                                    |
|  - Cases   |  Case list  |  Alert list  |  Observable list     |
|  - Alerts  |                                                    |
|  - Tasks   |  Statistics and metrics                          |
|  - Metrics |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Status Bar  |  Connected  |  User  |  Notifications           |
+------------------------------------------------------------------+
```

### 4.2 Key Components

- **Cases:** Security incidents being investigated
- **Alerts:** Notifications from security tools
- **Observables:** IoCs and artifacts
- **Tasks:** Response activities
- **Metrics:** Performance statistics

---

## 5. Case Management

### 5.1 Creating Cases

```
Cases > Create Case > Fill details > Save
```

### 5.2 Case Fields

| Field | Description |
|-------|-------------|
| Title | Case name |
| Description | Incident details |
| Severity | Critical/High/Medium/Low |
| TLP | Traffic Light Protocol |
| Tags | Categorization tags |
| Assignee | Team member responsible |

### 5.3 Case Lifecycle

```
New → In Progress → Resolved → Closed
```

### 5.4 Case Templates

Create templates for common incident types:
- Phishing Campaign
- Malware Infection
- Data Breach
- DDoS Attack
- Unauthorized Access

---

## 6. Alert Analysis

### 6.1 Alert Sources

| Source | Description |
|--------|-------------|
| Email | Phishing reports |
| SIEM | Security alerts |
| IDS/IPS | Network alerts |
| EDR | Endpoint alerts |
| Manual | User reports |
| MISP | Threat intelligence |

### 6.2 Processing Alerts

1. **Review:** Examine alert details
2. **Triage:** Determine if actionable
3. **Create Case:** If investigation needed
4. **Add Observables:** Extract IoCs
5. **Analyze:** Use Cortex for analysis
6. **Respond:** Take action

### 6.3 Alert Workflow

```
New → Under Review → Ignored | Created Case
```

---

## 7. Observable Analysis

### 7.1 Observable Types

| Type | Description |
|------|-------------|
| IP Address | Network addresses |
| Domain | Domain names |
| URL | Web addresses |
| Email | Email addresses |
| File | Malware samples |
| Hash | File hashes |

### 7.2 Adding Observables

```
Case > Observables > Add Observable > Fill details > Save
```

### 7.3 Analyzing Observables

1. Add observable to case
2. Run Cortex analyzers
3. Review results
4. Add to MISP (if threat)
5. Document findings

---

## 8. Task Management

### 8.1 Creating Tasks

```
Case > Tasks > Create Task > Fill details > Save
```

### 8.2 Task Fields

| Field | Description |
|-------|-------------|
| Title | Task name |
| Description | Task details |
| Status | To Do/In Progress/Done |
| Assignee | Team member |
| Due Date | Deadline |

### 8.3 Task Workflow

```
To Do → In Progress → Done
```

---

## 9. Cortex Integration

### 9.1 What is Cortex?

Cortex is TheHive's analysis and response engine that:
- Analyzes observables automatically
- Executes response actions
- Provides threat intelligence
- Integrates with multiple services

### 9.2 Configuring Cortex

```
Administration > Cortex > Add Server > Configure
```

### 9.3 Available Analyzers

| Analyzer | Description |
|----------|-------------|
| VirusTotal | Malware analysis |
| Shodan | IP reconnaissance |
| AbuseIPDB | IP reputation |
| MaxMind | GeoIP lookup |
| YARA | Malware detection |

---

## 10. MISP Integration

### 10.1 What is MISP?

MISP (Malware Information Sharing Platform) is a threat intelligence platform for sharing IoCs.

### 10.2 Configuring MISP

```
Administration > MISP > Add Server > Configure
```

### 10.3 Using MISP

- Import events from MISP
- Export observables to MISP
- Sync IoCs automatically
- Share findings with community

---

## 11. Metrics and Reporting

### 11.1 Built-in Metrics

- Total cases
- Open cases
- Mean time to resolve
- Cases by severity
- Alert statistics

### 11.2 Dashboard Creation

```
Dashboards > Create Dashboard > Add widgets > Save
```

### 11.3 Custom Reports

```
Cases > Select case > Generate Report > Configure > Generate
```

---

## 12. API Usage

### 12.1 Authentication

```bash
# API key from UI
# Administration > Users > API Key

# Use in requests
curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:9000/api/v1/
```

### 12.2 Common API Endpoints

```bash
# List cases
curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:9000/api/v1/case

# Create case
curl -X POST http://localhost:9000/api/v1/case \
    -H "Authorization: Bearer YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{"title":"New Incident","severity":"High"}'

# List alerts
curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:9000/api/v1/alert
```

---

## 13. Best Practices

### 13.1 Incident Response Workflow

1. **Detection:** Receive alert
2. **Triage:** Assess severity
3. **Investigation:** Create case, add observables
4. **Containment:** Isolate affected systems
5. **Eradication:** Remove threat
6. **Recovery:** Restore systems
7. **Lessons Learned:** Document and improve

### 13.2 Case Organization

- Use consistent naming conventions
- Tag cases by type and severity
- Assign clear ownership
- Document all actions
- Update status regularly

### 13.3 Team Collaboration

- Use tasks to assign work
- Comment on case updates
- Review each other's work
- Share findings via MISP

---

## 14. Troubleshooting

### Common Issues

**Problem:** TheHive won't start

```bash
# Check logs
docker logs thehive

# Check Elasticsearch
curl http://localhost:9200

# Check Cassandra
nodetool status
```

**Problem:** Cannot connect to database

```bash
# Check database status
systemctl status elasticsearch
systemctl status cassandra

# Restart databases
sudo systemctl restart elasticsearch
sudo systemctl restart cassandra
```

**Problem:** Cortex not working

```bash
# Check Cortex status
docker logs cortex

# Verify configuration
curl http://localhost:9001/api/status

# Restart Cortex
docker restart cortex
```

---

## References

- [TheHive Official Website](https://thehive-project.org/)
- [TheHive GitHub](https://github.com/TheHive-Project/TheHive)
- [TheHive Documentation](https://docs.strangebee.com/thehive/)
- [Cortex Documentation](https://docs.strangebee.com/cortex/)
- [MISP Project](https://www.misp-project.org/)
