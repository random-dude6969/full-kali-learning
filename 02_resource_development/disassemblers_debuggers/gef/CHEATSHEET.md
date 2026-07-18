# GEF (GDB Enhanced Features) Cheatsheet

## Quick Reference Card

### Installation

```bash
# Kali Linux
sudo apt install gef

# One-line install
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"

# Manual install
wget -O ~/.gdbinit-gef.py -q https://gef.blah.cat/py
echo "source ~/.gdbinit-gef.py" >> ~/.gdbinit
```

### Update

```
gef> gef update
```

### Context Display

GEF automatically shows after every stopped event:

```
┌─────────────────────── Registers ───────────────────────┐
│ rax   : 0x0           rbx   : 0x0                      │
│ rcx   : 0x0           rdx   : 0x7fffffffe288           │
│ rdi   : 0x1           rsi   : 0x7fffffffe278           │
│ ...                                                      │
└─────────────────────────────────────────────────────────┘
┌──────────────────── Code (x86-64) ─────────────────────┐
│ 0x401234  mov   eax, 0                                  │
│ 0x401239  pop   rbp                                     │
│ ►0x40123a  ret                        ← $rip            │
└─────────────────────────────────────────────────────────┘
┌─────────────────────── Stack ───────────────────────────┐
│ 0x7fffffffe1f8│ 0x00007ffff7e3a831 │___libc_start_main │
│ 0x7fffffffe200│ 0x0000000000000000 │                    │
└─────────────────────────────────────────────────────────┘
┌──────────────────── Memory Map ────────────────────────┐
│ 0x0000000000400000 0x0000000000401000 r-- /path/to/bin │
│ 0x0000000000401000 0x0000000000402000 r-x /path/to/bin │
└─────────────────────────────────────────────────────────┘
```

### Essential GEF Commands

| Command | Description |
|---------|-------------|
| `checksec` | Check binary protections (NX, PIE, Canary, RELRO, Fortify) |
| `vmmap` | Display memory mappings |
| `search` | Search memory |
| `dereference` | Dereference pointers from address |
| `heap` | Heap analysis commands |
| `heap chunks` | List all heap chunks |
| `heap bins` | Show free lists |
| `got` | Display Global Offset Table |
| `plt` | Display Procedure Linkage Table |
| `pattern create <len>` | Create de Bruijn pattern |
| `pattern search <val>` | Find pattern offset |
| `aslr` | Show/toggle ASLR status |
| `canary` | Show stack canary value |
| `telescope <addr> <count>` | Dereference chain viewer |
| `edit -f <file>` | Edit file |
| `kami` | Hide GEF from debugger detection |
| `hijack` | Hijack breakpoint for syscall |
| `trace` | Syscall tracing |
| `got` | Show GOT entries |
| `entry` | Show entry points |
| `EIF` | Extract INFOrmation (binary info) |
| `elf-info` | ELF header information |

### Binary Protection Checks

```
gef> checksec
    Arch:     amd64-64-little
    RELRO:    Full RELRO          # GOT is read-only
    Stack:    Canary found         # Stack canaries enabled
    NX:       NX enabled           # Non-executable stack
    PIE:      PIE enabled          # Position Independent Executable
    FORTIFY:  Fortified            # FORTIFY_SOURCE enabled
```

### Memory Search

```bash
# Search for string
gef> search "/bin/sh"
gef> search "AAAA"

# Search for pointer
gef> search -p 0x41414141

# Search in specific region
gef> search -p 0xdeadbeef -range 0x7fff0000-0x7fff1000
```

### Memory Map Inspection

```
gef> vmmap
             Start                End                Offset   Perm Path
0x0000555555554000 0x0000555555555000 0x00000000 r-- /path/to/binary
0x0000555555555000 0x0000555555559000 0x00001000 r-x /path/to/binary
0x00007ffff7dc3000 0x00007ffff7de8000 0x00000000 r-- /usr/lib/libc.so.6
0x00007ffff7de8000 0x00007ffff7f38000 0x00025000 r-x /usr/lib/libc.so.6
0x00007ffffffde000 0x00007ffffffff000 0x00000000 rw- [stack]
```

### Heap Analysis

```
# List all chunks
gef> heap chunks

# Show specific arena
gef> heap chunks --arena 0x7ffff7fb1b80

# Show tcache bins
gef> heap bins tcache

# Show unsorted bin
gef> heap bins unsorted

# Show fastbins
gef> heap bins fast
```

### Pattern Tools

```
# Create pattern
gef> pattern create 200
# Output: Aa0Aa1Aa2Aa3... (200 bytes)

# Search for pattern value
gef> pattern search 0x41366241
# Output: Found at offset 72
```

### GOT/PLT Inspection

```
gef> got
gef> plt
```

### Dereference Chain

```
gef> dereference $rsp 4
# Shows 4 pointer-sized values starting from RSP
```

### ASLR Control

```
gef> aslr
ASLR is currently: Enabled

gef> aslr off
gef> aslr on
```

### GEF Configuration

```
# Show all settings
gef>gef config

# Set specific config
gef>gef config context.enable 1
gef>gef context.redirect /dev/null    # Hide context for speed

# Enable debug mode
gef>gef config gef.debug 1
```

### Custom GEF Command Template

```python
import gdb

class MyCommand(gdb.Command):
    """Description of my command"""
    def __init__(self):
        super().__init__("my-cmd", gdb.COMMAND_USER)

    def invoke(self, arg, from_tty):
        # Your logic here
        gdb.execute("info registers", to_string=True)

MyCommand()
```

### Workflow: Exploit Development

```bash
# 1. Check protections
gef> checksec

# 2. Find overflow offset
gef> pattern create 200
gef> run <<< $(pattern 200)
gef> pattern search $rip

# 3. Find /bin/sh in memory
gef> search "/bin/sh"

# 4. Check memory layout
gef> vmmap

# 5. Inspect GOT
gef> got

# 6. Build exploit with offset + addresses
```

### GEF Extras (Extended Commands)

```bash
# Install extras
wget -O /tmp/gef-extras.py https://raw.githubusercontent.com/hugsy/gef-extras/main/gef.py
source /tmp/gef-extras.py
```

### Quick Reference: GDB + GEF Commands

| Task | Command |
|------|---------|
| Start | `run` |
| Continue | `continue` |
| Step into | `stepi` |
| Step over | `nexti` |
| Breakpoint | `break *0x401234` or `break main` |
| Backtrace | `backtrace` |
| Print register | `print $rax` |
| Set register | `set $rax = 0` |
| Examine memory | `x/10x $rsp` |
| Search memory | `search "string"` |
| Check security | `checksec` |
| Memory map | `vmmap` |
| Heap chunks | `heap chunks` |
| Pattern create | `pattern create 200` |
| Pattern search | `pattern search 0x41414141` |
