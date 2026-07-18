# Radare2 Cheatsheet

## Quick Reference Card

### Installation

```bash
# Kali Linux
sudo apt install radare2

# From source (recommended for latest)
git clone https://github.com/radareorg/radare2
cd radare2 && sys/install.sh
```

### Launch

```bash
r2 /path/to/binary                       # Open binary
r2 -A /path/to/binary                    # Open + auto-analyze
r2 -w /path/to/binary                    # Open in write mode
r2 -q -c 'aaa; afl' /path/to/binary     # Script mode
r2 -D gdb gdb://192.168.1.100:1234       # Remote debugging
```

### Essential Commands

| Command | Description |
|---------|-------------|
| `aaa` | Analyze all (functions, symbols, strings) |
| `aaaa` | Deep analysis (slower, more thorough) |
| `afl` | List all functions |
| `aflm` | List functions (minimal) |
| `pdf @main` | Disassemble function at main |
| `axt @main` | Cross-references to main |
| `axf @main` | Cross-references from main |
| `iz` | List strings |
| `izz` | List strings (all sections) |
| `ii` | List imports |
| `ie` | List entry points |
| `iS` | List sections |
| `is` | List symbols |
| `px 64` | Hex dump 64 bytes |
| `pxA 64` | Hex dump with ASCII art |
| `s main` | Seek to main |
| `s+` / `s-` | Seek history forward/back |
| `?` | Help |
| `q` | Quit |

### Seeking

| Command | Description |
|---------|-------------|
| `s addr` | Seek to address |
| `s sym.main` | Seek to symbol |
| `s+` | Undo seek |
| `s-` | Redo seek |
| `s.` | Show current seek |
| `sr addr` | Seek and register in history |
| `sl [+-]N` | Seek left/right N bytes |

### Configuration

| Command | Description |
|---------|-------------|
| `e` | List all config vars |
| `e asm.arch` | Show/set architecture |
| `e asm.bits` | Show/set bit width |
| `e scr.color` | Enable/disable colors |
| `e asm.syntax=intel` | Set Intel syntax |
| `e asm.emu=true` | Enable emulation |

### Assembly/Disassembly

```bash
# Disassemble
rasm2 -d 554889e5                      # Disassemble hex
rasm2 -a x86 -b 64 -d 554889e5        # Explicit arch

# Assemble
rasm2 -a x86 -b 64 'push rbp'         # Assemble to hex

# In r2 interactive mode:
pd 20                                  # Disassemble 20 instructions
pdr                                   # Disassemble function
pda                                   # Disassemble all (include data)
```

### Binary Info (rabin2)

```bash
rabin2 -I binary                       # Binary info
rabin2 -s binary                       # Symbols
rabin2 -i binary                       # Imports
rabin2 -z binary                       # Strings
rabin2 -S binary                       # Sections
rabin2 -l binary                       # Libraries
rabin2 -H binary                       # Header fields
rabin2 -c binary                       # Classes
```

### Hashing (rahash2)

```bash
rahash2 binary                         # Default hash (sha256)
rahash2 -a md5 binary                  # MD5 hash
rahash2 -a md5,sha1,sha256 binary      # Multiple hashes
rahash2 -s "string" -a sha256          # Hash string
```

### Searching (rafind2)

```bash
rafind2 -s "string" binary             # Search string
rafind2 -x 90909090 binary             # Search hex pattern
rafind2 -z binary                      # Find zero-terminated strings
rafind2 -e "regex" binary              # Regex search
```

### Binary Diffing (radiff2)

```bash
radiff2 old.exe new.exe                # Basic diff
radiff2 -s old.exe new.exe             # Edit distance
radiff2 -C old.exe new.exe             # Code diff (graph)
```

### Shellcode Generation (ragg2)

```bash
ragg2 -a x86 -b 32 -f python execve   # x86 shellcode
ragg2 -a x86 -b 64 -f C execve        # x64 C format
ragg2 -L                              # List available shellcodes
```

### Scripting (r2pipe Python)

```python
import r2pipe
r2 = r2pipe.open("/bin/ls")
r2.cmd("aaa")
functions = r2.cmdj("aflj")
strings = r2.cmdj("izj")
r2.quit()
```

### Pipe/Filters

| Syntax | Description |
|--------|-------------|
| `~keyword` | Filter output (grep) |
| `~...` | Interactive filter |
| `@@ addr` | Iterate over addresses |
| `;` | Chain commands |
| `j` suffix | JSON output |
| `r` suffix | Rizin output format |

### Debugging

| Command | Description |
|---------|-------------|
| `d` | Debug mode commands |
| `dc` | Continue debugging |
| `ds` | Step one instruction |
| `dso` | Step over |
| `db addr` | Set breakpoint |
| `dr` | Show registers |
| `dr rax=0` | Set register |
| `dbt` | Backtrace |
| `dt addr` | Trace execution |
| `dp` | List processes |

### Common Workflows

```bash
# Quick malware triage
r2 -A -q -c 'aaa; afl; iz; iS' suspicious.exe

# Find dangerous imports
r2 -q -c 'ii~gets; ii~strcpy; ii~scanf' binary

# Extract strings and check for IOCs
r2 -q -c 'iz' binary | grep -i "http\|ftp\|tcp"

# Patch binary (NOP a function)
r2 -w -q -c 's sym.vuln_func; wa nop; wa nop; wa nop; wa nop; q' binary
```

### Companion Tools

| Tool | Purpose |
|------|---------|
| `r2` | Main framework |
| `rabin2` | Binary info extractor |
| `rasm2` | Assembler/disassembler |
| `rahash2` | Hashing utility |
| `radiff2` | Binary diffing |
| `rafind2` | Pattern search |
| `ragg2` | Shellcode generator |
| `rarun2` | Program execution wrapper |
| `r2pm` | Package manager |
