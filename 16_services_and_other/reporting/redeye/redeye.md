# RedEye - Reconnaissance and Visualization Tool

## Complete Educational Guide for Security Professionals

---

## 1. Introduction

RedEye is a reconnaissance visualization and analysis tool designed to help security professionals analyze and visualize data collected during reconnaissance phases of penetration testing. It provides a visual interface for mapping network topologies, analyzing relationships between discovered hosts, and identifying attack paths.

### 1.1 What is RedEye?

RedEye is an open-source tool that:
- Visualizes reconnaissance data
- Maps network relationships
- Analyzes host connectivity
- Identifies potential attack paths
- Provides interactive graph visualization
- Supports import from multiple tools

### 1.2 Project Information

| Property | Value |
|----------|-------|
| Type | Reconnaissance Visualization |
| License | Open Source |
| Language | Python |
| Interface | Web-based |
| Database | SQLite |
| GitHub | observed-adversary/red-eye |

### 1.3 Why Use RedEye for Reconnaissance?

- **Visual Analysis:** See relationships between hosts
- **Pattern Detection:** Identify network patterns
- **Attack Path Mapping:** Find potential attack vectors
- **Data Correlation:** Combine data from multiple tools
- **Documentation:** Create visual evidence
- **Team Collaboration:** Share recon findings

### 1.4 RedEye Features

| Feature | Description |
|---------|-------------|
| Graph Visualization | Interactive network graphs |
| Multi-tool Import | Nmap, Nessus, Burp, etc. |
| Relationship Mapping | Host connections and dependencies |
| Attack Path Analysis | Identify potential attack vectors |
| Filtering | Filter by hosts, services, ports |
| Export | Generate reports and images |

---

## 2. Installation

### 2.1 Kali Linux

```bash
# Install via pip
pip3 install redeye

# Or from source
git clone https://github.com/observed-adversary/red-eye.git
cd red-eye
pip3 install -r requirements.txt

# Launch
python3 -m redeye
```

### 2.2 Docker

```bash
# Pull and run
docker pull redeye/redeye
docker run -p 8080:8080 redeye/redeye

# Access at http://localhost:8080
```

### 2.3 From Source

```bash
# Clone repository
git clone https://github.com/observed-adversary/red-eye.git
cd red-eye

# Install dependencies
pip3 install -r requirements.txt

# Initialize database
python3 -m redeye init

# Launch server
python3 -m redeye server

# Access at http://localhost:8080
```

### 2.4 First-Time Setup

1. Launch RedEye
2. Create a new project
3. Import reconnaissance data
4. Start analyzing

---

## 3. Interface Overview

### 3.1 Web Interface

```
+------------------------------------------------------------------+
|  RedEye  |  Projects  |  Import  |  Analysis  |  Export          |
+------------------------------------------------------------------+
|            |                                                    |
|  Sidebar   |              Graph Area                            |
|            |                                                    |
|  - Hosts   |  Interactive visualization of hosts and services  |
|  - Services|                                                    |
|  - Ports   |  Drag, zoom, filter                               |
|  - Filters |                                                    |
|            |                                                    |
+------------------------------------------------------------------+
|  Details Panel  |  Host Info  |  Service Details               |
+------------------------------------------------------------------+
```

### 3.2 Key Components

- **Project Management:** Create and manage recon projects
- **Import Panel:** Import data from various tools
- **Graph View:** Visual representation of network
- **Detail Panel:** Information about selected items
- **Filter Panel:** Filter displayed data
- **Export Panel:** Generate reports

---

## 4. Importing Reconnaissance Data

### 4.1 Supported Tools

| Tool | Import Format |
|------|---------------|
| Nmap | XML output |
| Nessus | XML export |
| Burp Suite | XML export |
| Masscan | XML output |
| OpenVAS | XML export |
| Custom | JSON/CSV |

### 4.2 Nmap Import

```bash
# Generate Nmap XML
nmap -sV -sC -p- -oX scan.xml target.com

# Import into RedEye
# UI: Import > Nmap XML > Upload scan.xml
```

### 4.3 Nessus Import

```bash
# Export from Nessus
# My Scans > Select > Export > Nessus

# Import into RedEye
# UI: Import > Nessus > Upload .nessus file
```

### 4.4 Multiple Import Sources

```bash
# Combine multiple scans
nmap -sV -oX nmap_scan.xml 192.168.1.0/24
masscan -p1-65535 -oX masscan_scan.xml 192.168.1.0/24

# Import both into RedEye
# Correlate findings
```

---

## 5. Graph Visualization

### 5.1 Graph Layout Options

- **Force-directed:** Automatic layout
- **Hierarchical:** Tree structure
- **Circular:** Circular arrangement
- **Grid:** Grid layout

### 5.2 Graph Interactions

- **Zoom:** Mouse wheel
- **Pan:** Click and drag
- **Select:** Click on node
- **Multi-select:** Ctrl+click
- **Drag:** Move nodes
- **Right-click:** Context menu

### 5.3 Node Types

| Node Type | Description |
|-----------|-------------|
| Host | Network host |
| Service | Running service |
| Port | Open port |
| Vulnerability | Security issue |

### 5.4 Edge Types

| Edge Type | Description |
|-----------|-------------|
| Connection | Network connection |
| Service | Service on host |
| Dependency | Service dependency |

---

## 6. Analysis Features

### 6.1 Host Analysis

```bash
# View host details
Click on host node > View details

# Filter by host
# Sidebar > Hosts > Select host
```

### 6.2 Service Analysis

```bash
# View services
Click on service node > View details

# Filter by service type
# Sidebar > Services > Select service
```

### 6.3 Port Analysis

```bash
# View ports
Click on port node > View details

# Filter by port number
# Sidebar > Ports > Enter port
```

### 6.4 Relationship Analysis

```bash
# View connections
Click on edge > View relationship

# Trace attack paths
# Follow edges from entry point
```

---

## 7. Attack Path Analysis

### 7.1 Identifying Attack Paths

1. Identify entry points (exposed services)
2. Trace connections to internal hosts
3. Identify service dependencies
4. Map potential attack vectors

### 7.2 Common Attack Paths

```
Internet → Web Server → Database
Internet → Mail Server → Internal Network
Internal → Domain Controller → All Hosts
```

### 7.3 Attack Path Visualization

- Use graph view to trace paths
- Filter by service type
- Identify critical nodes
- Document attack scenarios

---

## 8. Data Correlation

### 8.1 Combining Data Sources

```bash
# Import Nmap scan
nmap -sV -oX nmap.xml target.com

# Import Nessus scan
nessus_scan.nessus

# Correlate in RedEye
# - Match hosts across scans
# - Combine service information
# - Identify gaps
```

### 8.2 Host Correlation

- Match by IP address
- Match by hostname
- Match by MAC address
- Match by service fingerprint

### 8.3 Service Correlation

- Match by port number
- Match by service name
- Match by version
- Match by banner

---

## 9. Filtering and Search

### 9.1 Filter Options

| Filter | Description |
|--------|-------------|
| Host | Filter by host IP/hostname |
| Service | Filter by service type |
| Port | Filter by port number |
| OS | Filter by operating system |
| Status | Filter by host status |

### 9.2 Search Features

- Search by host name
- Search by IP address
- Search by service name
- Search by port number
- Search by vulnerability

---

## 10. Export Capabilities

### 10.1 Export Formats

| Format | Description |
|--------|-------------|
| PNG | Graph image |
| SVG | Vector image |
| PDF | Document |
| JSON | Data export |
| CSV | Spreadsheet |

### 10.2 Generating Reports

```
Export > Report > Select format > Configure > Generate
```

### 10.3 Export Options

- Full graph
- Filtered view
- Host list
- Service list
- Attack paths

---

## 11. Best Practices

### 11.1 Reconnaissance Workflow

1. **Collect:** Gather data from multiple tools
2. **Import:** Bring data into RedEye
3. **Correlate:** Match hosts and services
4. **Analyze:** Identify patterns and relationships
5. **Document:** Export findings
6. **Report:** Create visual documentation

### 11.2 Data Organization

- Use descriptive project names
- Tag important hosts
- Document attack paths
- Save filtered views

### 11.3 Team Collaboration

- Share projects
- Document assumptions
- Validate findings
- Review together

---

## 12. Integration with Other Tools

### 12.1 Nmap Workflow

```bash
# 1. Scan with Nmap
nmap -sV -sC -p- -oX scan.xml target.com

# 2. Import into RedEye
# 3. Analyze graph
# 4. Export findings
# 5. Document in CherryTree/Dradis
```

### 12.2 Nessus Workflow

```bash
# 1. Run Nessus scan
# 2. Export results
# 3. Import into RedEye
# 4. Correlate with Nmap data
# 5. Generate report
```

### 12.3 Combined Workflow

```bash
# Complete recon workflow
nmap -sV -sC -p- -oX nmap.xml target.com
nikto -h target.com -o nikto.html
# Import all into RedEye
# Analyze and correlate
# Generate comprehensive report
```

---

## 13. Troubleshooting

### Common Issues

**Problem:** Server won't start

```bash
# Check Python version
python3 --version

# Install dependencies
pip3 install -r requirements.txt

# Check port availability
netstat -tlnp | grep 8080
```

**Problem:** Import errors

```bash
# Verify file format
# Check XML structure
# Ensure all required fields present
```

**Problem:** Graph not rendering

```bash
# Check browser compatibility
# Clear browser cache
# Try different browser
```

---

## 14. Advanced Analysis Techniques

### 14.1 Network Topology Mapping

RedEye excels at visualizing network topology:

```
Import > Nmap XML > Analyze > Export Topology
```

### 14.2 Host Relationship Analysis

```bash
# Identify host relationships
# - Shared services
# - Network segments
# - Trust relationships
# - Communication patterns
```

### 14.3 Attack Surface Mapping

```
1. Import all scans
2. Identify exposed services
3. Map external to internal
4. Document attack vectors
```

---

## 15. Data Enrichment

### 15.1 Adding External Data

```bash
# Import additional data sources
# - WHOIS data
# - DNS records
# - Certificate information
# - Geolocation data
```

### 15.2 Correlating Data

```bash
# Match hosts across scans
# - IP address correlation
# - Hostname matching
# - Service fingerprint matching
# - Port number correlation
```

### 15.3 Creating Custom Fields

```bash
# Add custom properties to hosts
# - Business criticality
# - Owner information
# - Location data
# - Compliance requirements
```

---

## 16. Export and Reporting

### 16.1 Graph Export

```bash
# Export graph as image
Export > Graph > PNG/SVG

# Export filtered view
Apply filters > Export > Select format
```

### 16.2 Data Export

```bash
# Export host list
Export > Hosts > CSV

# Export service list
Export > Services > CSV

# Export full data
Export > All Data > JSON
```

### 16.3 Report Generation

```bash
# Generate report
Reports > Create Report > Select sections > Generate

# Report sections
- Executive Summary
- Network Topology
- Host Details
- Service Inventory
- Attack Paths
```

---

## 17. Best Practices

### 17.1 Data Organization

- Use consistent naming conventions
- Tag important hosts
- Document assumptions
- Save filtered views
- Export regularly

### 17.2 Analysis Workflow

1. **Import:** Bring in all scan data
2. **Correlate:** Match hosts across sources
3. **Analyze:** Identify patterns
4. **Document:** Add notes and tags
5. **Export:** Generate reports

### 17.3 Team Collaboration

- Share projects
- Assign analysis tasks
- Review each other's work
- Document methodology

---

## References

- [RedEye GitHub](https://github.com/observed-adversary/red-eye)
- [RedEye Documentation](https://github.com/observed-adversary/red-eye/wiki)
- [Nmap Documentation](https://nmap.org/docs.html)
- [Kali Linux Tools](https://www.kali.org/tools/)
