# Rizin Cheatsheet

## Quick Reference Card

### Installation

```bash
# Kali Linux
sudo apt install rizin

# From source
git clone https://github.com/rizinorg/rizin
cd rizin && meson setup build
meson compile -C build && sudo meson install -C build
```

### Launch

```bash
rizin /path/to/binary                   # Open binary
rizin -A /path/to/binary               # Open + auto-analyze
rizin -w /path/to/binary               # Open in write mode
rizin -q -c 'aaa; afl' /path/to/binary # Script mode
rizin -D gdb gdb://192.168.1.100:1234  # Remote debugging
```

### Essential Commands

| Command | Description |
|---------|-------------|
| `aaa` | Analyze all (functions, symbols, strings) |
| `aaaa` | Deep analysis |
| `afl` | List all functions |
| `pdf @main` | Disassemble at main |
| `axt @main` | Cross-references to main |
| `axf @main` | Cross-references from main |
| `iz` | List strings |
| `ii` | List imports |
| `ie` | List entry points |
| `iS` | List sections |
| `is` | List symbols |
| `px 64` | Hex dump 64 bytes |
| `s main` | Seek to main |
| `s+` / `s-` | Seek history forward/back |
| `q` | Quit |

### Seeking

| Command | Description |
|---------|-------------|
| `s addr` | Seek to address |
| `s sym.main` | Seek to symbol |
| `s+` | Undo seek |
| `s-` | Redo seek |
| `s.` | Show current seek |

### Configuration

| Command | Description |
|---------|-------------|
| `e` | List all config vars |
| `e asm.arch` | Show/set architecture |
| `e asm.bits` | Show/set bit width |
| `e scr.color` | Enable/disable colors |
| `e asm.syntax=intel` | Intel syntax |

### Assembly/Disassembly (rz-asm)

```bash
# Standalone
rz-asm -d 554889e5                     # Disassemble hex
rz-asm -a x86 -b 64 -d 554889e5       # Explicit arch
rz-asm -a x86 -b 64 'push rbp'        # Assemble to hex

# In interactive mode
pd 20                                  # Disassemble 20 instructions
pdr                                   # Disassemble function
```

### Binary Info (rz-bin)

```bash
rz-bin -I binary                       # Binary info
rz-bin -s binary                       # Symbols
rz-bin -i binary                       # Imports
rz-bin -z binary                       # Strings
rz-bin -S binary                       # Sections
rz-bin -l binary                       # Libraries
rz-bin -H binary                       # Header fields
rz-bin -e binary                       # Entry points
```

### Hashing (rz-hash)

```bash
rz-hash binary                         # Default hash
rz-hash -a md5 binary                  # MD5
rz-hash -a md5,sha1,sha256 binary      # Multiple hashes
rz-hash -s "string" -a sha256          # Hash string
rz-hash -E aes -K key -I iv binary    # Encryption
```

### Searching (rz-find)

```bash
rz-find -s "string" binary             # Search string
rz-find -x 90909090 binary             # Search hex pattern
rz-find -z binary                      # Find zero-terminated strings
rz-find -e "regex" binary              # Regex search
rz-find -w "wide_string" binary        # Search wide string
```

### Binary Diffing (rz-diff)

```bash
rz-diff old.exe new.exe                # Basic diff
rz-diff -A old.exe new.exe             # Full analysis diff
rz-diff -t functions old.exe new.exe   # Compare functions
rz-diff -t strings old.exe new.exe     # Compare strings
rz-diff -d leven old.exe new.exe       # Levenshtein distance
```

### Shellcode Generation (rz-gg)

```bash
rz-gg -a x86 -b 32 -f python execve   # x86 Python format
rz-gg -a x86 -b 64 -f C execve        # x64 C format
rz-gg -L                              # List shellcodes
rz-gg -i execve -a x86 -x             # Execute shellcode
```

### Base Conversion (rz-ax)

```bash
rz-ax 10                              # Decimal to hex
rz-ax 0xa                             # Hex to decimal
rz-ax -e 0x41                         # Swap endianness
rz-ax -E "hello"                      # Base64 encode
rz-ax -D "aGVsbG8="                  # Base64 decode
rz-ax -s 41414141                     # Hex string to raw
rz-ax -S < binary                     # Raw to hex string
rz-ax -i 3530468537                   # IP to long
rz-ax -I 3530468537                   # Long to IP
```

### Scripting (rzpipe Python)

```python
import rzpipe

r = rzpipe.open("/usr/bin/ls")
r.cmd("aaa")

# JSON output
functions = r.cmdj("aflj")
strings = r.cmdj("izj")
imports = r.cmdj("iij")

for func in functions:
    print(f"{func['name']}: 0x{func['offset']:x}")

r.quit()
```

### Pipe/Filters

| Syntax | Description |
|--------|-------------|
| `~keyword` | Filter output (grep) |
| `;` | Chain commands |
| `j` suffix | JSON output |
| `r` suffix | Rizin output format |
| `@@ addr` | Iterate over addresses |

### Debugging

| Command | Description |
|---------|-------------|
| `dc` | Continue |
| `ds` | Step one instruction |
| `dso` | Step over |
| `db addr` | Set breakpoint |
| `dr` | Show registers |
| `dr rax=0` | Set register |
| `dbt` | Backtrace |
| `dp` | List processes |

### FLIRT Signatures (rz-sign)

```bash
rz-sign -o libc.sig /lib/libc.so.6    # Create signature
rz-sign -d signature.sig              # Dump signature
rz-sign -c output.pat input.sig       # Convert format
```

### Package Management (rizipm)

```bash
rizipm -s keyword                     # Search plugins
rizipm -ci rz-ghidra                  # Install plugin
rizipm -l                             # List installed
rizipm -u plugin                      # Uninstall plugin
```

### Common Workflows

```bash
# Quick malware triage
rizin -A -q -c 'aaa; afl; iz; iS' suspicious.exe

# Find dangerous imports
rizin -q -c 'ii~gets; ii~strcpy; ii~scanf' binary

# Extract strings for IOCs
rizin -q -c 'iz' binary | grep -i "http\|ftp\|tcp"

# Patch binary (NOP a function)
rizin -w -q -c 's sym.vuln_func; wa nop; wa nop; wa nop; q' binary
```

### Companion Tools

| Tool | Purpose |
|------|---------|
| `rizin` | Main framework |
| `rz-bin` | Binary info extractor |
| `rz-asm` | Assembler/disassembler |
| `rz-hash` | Hashing utility |
| `rz-diff` | Binary diffing |
| `rz-find` | Pattern search |
| `rz-gg` | Shellcode generator |
| `rz-run` | Program execution wrapper |
| `rz-sign` | FLIRT signature tool |
| `rz-ax` | Base converter |
| `rizipm` | Package manager |
