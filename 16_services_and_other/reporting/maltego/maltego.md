# Maltego - OSINT and Link Analysis Tool

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

Maltego is a powerful open-source intelligence (OSINT) and forensics application that provides mining and gathering of information as well as representation of this information in an easy-to-understand graphical format. It is one of the most widely used tools for information gathering, network reconnaissance, and relationship mapping in cybersecurity.

### 1.1 What is Maltego?

Maltego is a graphical link analysis tool that:
- Gathers and visualizes relationships between data points
- Integrates with multiple OSINT data sources
- Provides interactive graph-based analysis
- Supports custom transforms and data sources
- Generates comprehensive reports

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Company | Maltego Technologies GmbH |
| License | Commercial (CE available) |
| Version | 4.11.3 |
| Language | Java |
| Platform | Linux, Windows, macOS |
| GitHub | maltego/maltego |
| Website | maltego.com |

### 1.3 Maltego CE vs Classic vs XL

| Feature | CE (Community) | Classic | XL |
|---------|----------------|---------|-----|
| Price | Free | Paid | Paid |
| Transforms | Limited | Full | Full |
| Entities | Limited | Full | Full |
| Save/Export | No | Yes | Yes |
| Commercial Use | No | Yes | Yes |

### 1.4 Why Use Maltego for OSINT?

- **Visual Analysis:** Graph-based relationship mapping
- **Multiple Data Sources:** Integrate with many OSINT providers
- **Automation:** Custom transforms for specific tasks
- **Interactive:** Drag-and-drop graph manipulation
- **Reporting:** Generate visual reports
- **Integration:** Connect with other security tools

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via apt
sudo apt update
sudo apt install maltego

# Launch Maltego
maltego
```

### 2.2 From Source

```bash
# Download from website
wget https://www.maltego.com/downloads/

# Extract
unzip maltego-*.zip
cd maltego-*

# Run
./maltego
```

### 2.3 Docker

```bash
# Run with Docker
docker run -it -p 5900:5900 maltego/maltego

# Connect via VNC
vnc://localhost:5900
```

### 2.4 First-Time Setup

1. Launch Maltego
2. Create an account or use Community Edition
3. Configure transforms
4. Start your first investigation

---

## 3. Interface Overview

### 3.1 Main Window Components

```
+------------------------------------------------------------------+
|  Menu Bar  |  File  Edit  View  Transform  Machines  Help        |
+------------------------------------------------------------------+
|  Toolbar   |  New  Open  Save  |  Run Transform  |  Zoom         |
+------------------------------------------------------------------+
|            |              Graph Area                             |
| Transform  |                                                    |
|  Results   |  Interactive graph with entities and edges          |
|            |                                                    |
| (Right)    |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Entity Palette  |  Detail View  |  Console                     |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **Graph Area:** Visual representation of entities and relationships
- **Transform Results:** Output of transform execution
- **Entity Palette:** Available entity types
- **Detail View:** Information about selected entity
- **Console:** Command output and logs

---

## 4. Entities

### 4.1 Entity Types

Maltego uses different entity types to represent data:

| Entity Type | Description |
|-------------|-------------|
| Person | Individual people |
| Email Address | Email addresses |
| Domain | Internet domains |
| IP Address | Network addresses |
| Phone Number | Phone numbers |
| Company | Organizations |
| Website | Web pages |
| Alias | Online usernames |
| Document | Files and documents |
| Social Network Profile | Social media accounts |

### 4.2 Creating Custom Entities

```
Manage > Entity Types > New Entity Type
```

---

## 5. Transforms

### 5.1 What are Transforms?

Transforms are operations that take an entity as input, query a data source, and return new entities as output.

### 5.2 Built-in Transforms

| Transform | Input | Output |
|-----------|-------|--------|
| To Domain | Domain | Subdomains, DNS info |
| To IP Address | Domain | IP addresses |
| To Person | Domain | Associated people |
| To Email | Domain | Email addresses |
| To Phone | Domain | Phone numbers |
| To Website | Domain | Linked websites |
| To Social Profile | Person | Social media |

### 5.3 Running Transforms

1. Right-click entity on graph
2. Select "Run Transform"
3. Choose specific transform
4. View results on graph

### 5.4 Bulk Transforms

Select multiple entities and run transforms on all:
```
Ctrl+A (select all) > Right-click > Run Transform
```

---

## 6. OSINT Techniques with Maltego

### 6.1 Domain Investigation

```
Start: example.com
├── To DNS Names → subdomains.example.com
├── To IP Address → 192.168.1.1
├── To MX Record → mail.example.com
├── To Whois Info → registrant information
└── To Person → associated individuals
```

### 6.2 Email Investigation

```
Start: user@example.com
├── To Domain → example.com
├── To Person → John Doe
├── To Social Profiles → LinkedIn, Twitter
├── To Phone Number → +1-555-1234
└── To Related Emails → other addresses
```

### 6.3 Network Investigation

```
Start: 192.168.1.0/24
├── To IP Addresses → All hosts
├── To Domains → Hostnames
├── To Open Ports → Services
├── To Websites → Web servers
└── To OS → Operating systems
```

### 6.4 Person Investigation

```
Start: John Doe
├── To Email Addresses → john@example.com
├── To Social Profiles → LinkedIn, Twitter
├── To Aliases → johndoe123
├── To Phone Numbers → +1-555-5678
└── To Related People → Colleagues
```

---

## 7. Custom Transforms

### 7.1 Installing Transforms

```
Transform Hub > Browse > Install Transform
```

### 7.2 Creating Custom Transforms

```python
# Example Python transform
from maltego_trx.maltego import MaltegoTransform
from maltego_trx.transform import DiscoverableTransform

class MyTransform(DiscoverableTransform):
    @classmethod
    def create_entities(cls, request, response):
        # Get input entity
        input_entity = request.Value
        
        # Query data source
        results = query_data_source(input_entity)
        
        # Add results to response
        for result in results:
            entity = response.addEntity("maltego.Person", result['name'])
            entity.addProperty("email", "Email", "loose", result['email'])
```

### 7.3 Local Transform Server

```bash
# Start transform server
python transform_server.py

# Register in Maltego
Manage > Transforms > Local Transforms > Add
```

---

## 8. Graph Analysis

### 8.1 Graph Layout

- **Hierarchical:** Tree-like structure
- **Organic:** Natural clustering
- **Circular:** Circular arrangement
- **Centrality:** Focus on connected nodes

### 8.2 Graph Operations

| Action | Description |
|--------|-------------|
| Add Entity | Drag from palette |
| Add Link | Connect two entities |
| Remove | Delete selected items |
| Group | Create entity group |
| Color | Change entity color |
| Expand | Show more connections |

### 8.3 Filtering

- Filter by entity type
- Filter by transform source
- Filter by connection count
- Search by text

---

## 9. Integration with Other Tools

### 9.1 Import from Tools

| Tool | Import Format |
|------|---------------|
| Nmap | XML |
| Nessus | XML |
| Burp Suite | XML |
| Metasploit | XML |
| Shodan | API |
| Censys | API |

### 9.2 Export to Tools

| Tool | Export Format |
|------|---------------|
| Dradis | XML |
| CherryTree | PNG/PDF |
| Report | PDF/HTML |

### 9.3 API Integration

```bash
# Use Maltego API for automation
# Connect with custom scripts
# Integrate with CI/CD pipelines
```

---

## 10. Report Generation

### 10.1 Creating Reports

```
Report > Generate Report > Select template > Configure > Generate
```

### 10.2 Report Templates

- Executive Summary
- Technical Findings
- Graph Visualization
- Entity List
- Transform Results

### 10.3 Export Options

| Format | Description |
|--------|-------------|
| PDF | Formal report |
| HTML | Web-ready |
| PNG | Graph image |
| CSV | Data export |
| GraphML | Graph data |

---

## 11. Best Practices

### 11.1 Investigation Workflow

1. **Plan:** Define investigation scope
2. **Gather:** Run initial transforms
3. **Analyze:** Review relationships
4. **Expand:** Run additional transforms
5. **Verify:** Validate findings
6. **Report:** Generate output

### 11.2 Graph Organization

- Use colors for entity types
- Group related entities
- Add notes to important nodes
- Clean up irrelevant connections

### 11.3 OSINT Ethics

- Respect privacy laws
- Get proper authorization
- Document methodology
- Protect sensitive data

---

## 12. Troubleshooting

### Common Issues

**Problem:** Maltego won't start

```bash
# Check Java version
java -version

# Set Java path
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

# Clear cache
rm -rf ~/.local/share/Maltego/
```

**Problem:** Transforms not working

```bash
# Check transform server
# Verify API keys
# Test connectivity
```

**Problem:** Slow performance

```bash
# Increase memory
# Edit maltego.ini
# Reduce graph complexity
```

---

## 13. Maltego Transform Development

### 13.1 Transform Architecture

Maltego transforms consist of:
- **Input:** Entity type and value
- **Processing:** Query data source
- **Output:** New entities with properties

### 13.2 Python Transform Example

```python
# transforms/MyTransform.py
from maltego_trx.maltego import MaltegoTransform, MaltegoMsg
from maltego_trx.transform import DiscoverableTransform

class ToIPAddresses(DiscoverableTransform):
    @classmethod
    def create_entities(cls, request: MaltegoMsg, response: MaltegoTransform):
        domain = request.Value
        import socket
        try:
            ips = socket.getaddrinfo(domain, None)
            for ip in ips:
                ip_addr = ip[4][0]
                entity = response.addEntity("maltego.IPv4Address", ip_addr)
                entity.addProperty("dns.name", "DNS Name", "loose", domain)
        except:
            pass
```

### 13.3 Transform Server

```python
# server.py
from maltego_trx.server import app
from maltego_trx.registry import register_transform

register_transform(ToIPAddresses)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

### 13.4 Registering Transforms

```
Manage > Transforms > Local Transforms > Add
Name: To IP Addresses
Command: python3 transforms/ToIPAddresses.py
Working Dir: /path/to/transforms
```

---

## 14. Advanced Graph Analysis

### 14.1 Graph Algorithms

Maltego supports various graph analysis techniques:

- **Centrality Analysis:** Identify important nodes
- **Clustering:** Group related entities
- **Path Finding:** Discover connections
- **Community Detection:** Find groups

### 14.2 Using Graph Views

```python
# View 1: All domains
# Run "To Domain" on seed entity

# View 2: Email relationships
# Run "To Email" on domains
# Run "To Person" on emails

# View 3: Network topology
# Run "To IP Address" on domains
# Run "To Open Ports" on IPs
```

### 14.3 Graph Export Options

| Format | Use Case |
|--------|----------|
| PNG | Static images |
| SVG | Scalable graphics |
| PDF | Reports |
| GraphML | Further analysis |
| CSV | Data processing |

---

## 15. Maltego CaseFile

### 15.1 What is CaseFile?

CaseFile is a lightweight version of Maltego focused on:
- Manual graph creation
- Evidence organization
- Relationship mapping
- No transforms required

### 15.2 CaseFile vs Maltego

| Feature | CaseFile | Maltego |
|---------|----------|---------|
| Transforms | No | Yes |
| Price | Free | Paid (CE available) |
| Use Case | Manual mapping | Automated recon |
| Complexity | Simple | Advanced |

### 15.3 CaseFile Use Cases

- Incident response documentation
- Evidence organization
- Relationship mapping
- Simple investigations

---

## 16. Best Practices for OSINT

### 16.1 Methodology

1. **Plan:** Define investigation scope
2. **Collect:** Gather data systematically
3. **Analyze:** Look for patterns
4. **Verify:** Validate findings
5. **Document:** Record everything
6. **Report:** Share results responsibly

### 16.2 Data Quality

- Verify information from multiple sources
- Document data provenance
- Note confidence levels
- Update findings as new info emerges
- Distinguish facts from assumptions

### 16.3 Legal Considerations

- Respect privacy laws (GDPR, CCPA)
- Get proper authorization
- Don't access systems without permission
- Protect collected data
- Document methodology

---

## 17. Maltego Shortcuts Reference

### 17.1 Graph Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + N | New graph |
| Ctrl + O | Open graph |
| Ctrl + S | Save graph |
| Ctrl + Z | Undo |
| Ctrl + Y | Redo |
| Ctrl + A | Select all |
| Delete | Remove selected |

### 17.2 Transform Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + T | Run transform |
| Ctrl + Shift + T | Run all transforms |
| F5 | Refresh |

### 17.3 View Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl + + | Zoom in |
| Ctrl + - | Zoom out |
| Ctrl + 0 | Reset zoom |
| Ctrl + F | Find entity |

---

## References

- [Maltego Official Website](https://www.maltego.com/)
- [Maltego Documentation](https://docs.maltego.com/)
- [Maltego Transform Hub](https://www.maltego.com/transform-hub/)
- [Kali Linux Maltego Package](https://www.kali.org/tools/maltego/)
- [Maltego CE Download](https://www.maltego.com/community-edition/)
- [Maltego Transform Development](https://docs.maltego.com/technical-documentation/developing-transforms/)
