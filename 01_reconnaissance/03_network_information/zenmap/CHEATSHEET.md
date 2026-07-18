# Zenmap — Cheatsheet

## Quick Start

```bash
# Launch Zenmap
zenmap

# Launch as root (for SYN scans)
sudo zenmap

# Quick scan from command line
zenmap -t "Intense scan" 192.168.1.1
```

## Built-in Scan Profiles

| Profile | Command |
|---------|---------|
| Intense scan | `nmap -T4 -A -v target` |
| Intense scan + UDP | `nmap -sS -sU -T4 -A -v target` |
| Intense scan, all TCP | `nmap -p 1-65535 -T4 -A -v target` |
| Intense scan, no ping | `nmap -T4 -A -v -Pn target` |
| Ping scan | `nmap -sn target` |
| Quick scan | `nmap -T4 -F target` |
| Quick scan plus | `nmap -T4 -O -sV target` |
| Regular scan | `nmap target` |
| Slow comprehensive | `nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389,445 -PB -O --script all target` |

## GUI Tabs

| Tab | Function |
|-----|----------|
| Nmap Output | Raw command output |
| Ports/Hosts | Port/service table |
| Topology | Network map visualization |
| Host Details | Per-host information |
| Scan Comparison | Compare multiple scans |

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl+N | New scan |
| Ctrl+O | Open scan results |
| Ctrl+S | Save scan results |
| Ctrl+P | Print results |
| Ctrl+F | Find in output |
| Ctrl+Q | Quit |
| F1 | Help |
| Ctrl+Shift+C | Copy |
| Ctrl+Shift+V | Paste |

## Topology Controls

| Action | Method |
|--------|--------|
| Zoom in/out | Mouse wheel |
| Pan | Click and drag |
| Select host | Click node |
| Host menu | Right-click node |
| Fit to screen | View → Fit |

## Display Filters

```
port 80              # Filter by port
service http         # Filter by service
host 192.168.1.1     # Filter by IP
state open           # Filter by state
port 80 and host 192.168.1.1  # Combined
```

## Custom Profile Creation

1. Profile → New Profile or Command
2. Configure tabs:
   - **Target:** Default targets
   - **Scan:** Scan type, timing
   - **Ping:** Host discovery
   - **Scripting:** NSE scripts
   - **Target Spec:** Exclusions
   - **User Interface:** Display
3. Save with descriptive name

## Workflow

```
1. Enter target → Select profile → Click Scan
2. Review Nmap Output tab
3. Check Ports/Hosts for services
4. Examine Topology for network layout
5. Use Host Details for specific systems
6. Compare with previous scans if needed
7. Export results (File → Save)
```

## Common Tasks

| Task | Steps |
|------|-------|
| Quick host discovery | Ping scan profile |
| Full port scan | Intense scan, all TCP |
| Service enumeration | Intense scan profile |
| OS detection | Quick scan plus profile |
| Change detection | Scan Comparison tab |
| Export for Metasploit | Save as XML → db_import |

## Command-Line Equivalents

```bash
# Launch with specific profile
zenmap -t "Intense scan" 192.168.1.1

# Open saved results
zenmap -r saved_scan.xml

# Quick scan from terminal
nmap -T4 -A -v 192.168.1.1
```
