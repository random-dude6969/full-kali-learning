# Sickle Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install sickle
# Or from source:
git clone https://github.com/wiregh0/sickle.git
cd sickle && pip3 install -r requirements.txt
```

### Format Conversion

```bash
# Hex to C array
sunny -s "9089c331d2" -f hex -o c

# C array to Python
sunny -s "\x90\x89\xc3\x31\xd2" -f c -o python

# Hex to raw binary
sunny -s "9089c331d2" -f hex -o raw -o shellcode.bin

# Python to hex
sunny -s "b'\x90\x89\xc3\x31\xd2'" -f python -o hex
```

### Bad Character Detection

```bash
# Check for default bad chars
sunny -s "9089c331d2000a" -f hex -c

# Check with custom list
sunny -s "9089c331d2000a0d" -f hex -c -b "\x00\x0a\x0d"
```

### Shell-Storm Integration

```bash
# Download by ID
sunny -S 837

# Download and convert to C
sunny -S 837 -f raw -o c

# Download and convert to Python
sunny -S 837 -f raw -o python

# Download and check bad chars
sunny -S 837 -f raw -c -b "\x00\x0a"
```

### Key Flags

| Flag | Description |
|------|-------------|
| `-s` | Shellcode input |
| `-f` | Source format: raw, hex, c, python, ruby |
| `-o` | Output format: raw, hex, c, python, ruby |
| `-c` | Check bad characters |
| `-b` | Custom bad char list |
| `-S` | Download from Shell-Storm |
| `-a` | Architecture: x86, x64 |
| `-v` | Verbose output |

### Pipeline Workflow

```bash
# 1. Get shellcode
sunny -S 837 -f raw -o hex > shell.hex

# 2. Check bad chars
sunny -s "$(cat shell.hex)" -f hex -c -b "\x00\x0a"

# 3. Convert for delivery
sunny -s "$(cat shell.hex)" -f hex -o python > shell.py
```

### Integration with Other Tools

```bash
# Feed to Shellter
sunny -S 837 -f raw -o raw > /tmp/shellcode.bin

# Convert for MSFvenom template
sunny -S 837 -f raw -o c
```

### Troubleshooting

```bash
# Shell-Storm download fails → manual download
wget http://shell-storm.org/api/shellcode/?id=837 -O shellcode.bin

# Invalid format → check -f matches input
sunny -s "9089c331d2" -f hex -o c  # Correct
sunny -s "\x90\x89" -f c -o hex    # Correct

# Missing dependencies
pip3 install requests
```
