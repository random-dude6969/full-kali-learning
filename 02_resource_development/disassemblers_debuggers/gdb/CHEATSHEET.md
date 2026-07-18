# GDB Cheatsheet

## Quick Reference Card

### Installation

```bash
sudo apt install gdb gdb-multiarch gdbserver
```

### Launch

```bash
gdb /path/to/binary                      # Debug a binary
gdb -q /path/to/binary                   # Quiet mode (no banner)
gdb -tui /path/to/binary                 # Text UI mode
gdb -batch -x script.gdb binary          # Batch mode with script
gdb-multiarch /path/to/arm_binary        # Multi-architecture
```

### Essential Commands

| Command | Short | Description |
|---------|-------|-------------|
| `run` | `r` | Start program |
| `run arg1 arg2` | `r arg1 arg2` | Start with arguments |
| `continue` | `c` | Continue to next breakpoint |
| `stepi` | `si` | Step one instruction (into calls) |
| `nexti` | `ni` | Step one instruction (over calls) |
| `step` | `s` | Step one source line (into functions) |
| `next` | `n` | Step one source line (over functions) |
| `finish` | `fin` | Run until current function returns |
| `quit` | `q` | Exit GDB |

### Breakpoints

| Command | Description |
|---------|-------------|
| `break main` | Break at function `main` |
| `break *0x401234` | Break at address |
| `break main+42` | Break at offset from function |
| `break file.c:25` | Break at source line |
| `break main if x==5` | Conditional breakpoint |
| `info breakpoints` | List all breakpoints |
| `delete 1` | Delete breakpoint #1 |
| `disable 1` | Disable breakpoint #1 |
| `enable 1` | Enable breakpoint #1 |
| `ignore 1 10` | Skip first 10 hits of BP#1 |

### Watchpoints

| Command | Description |
|---------|-------------|
| `watch var` | Break when var is written |
| `rwatch var` | Break when var is read |
| `awatch var` | Break on read or write |

### Examining Registers

| Command | Description |
|---------|-------------|
| `info registers` | Show all registers |
| `info registers eax ebx` | Show specific registers |
| `print $eax` | Print register value |
| `print $eax + 1` | Print expression |
| `set $eax = 0x0` | Modify register |

### Examining Memory

| Command | Description |
|---------|-------------|
| `x/10x $rsp` | 10 hex words from RSP |
| `x/20i $rip` | 20 instructions from RIP |
| `x/s 0x402000` | String at address |
| `x/40wx $rsp` | 40 hex words (stack) |
| `x/10i main` | 10 instructions at main |
| `x/x &var` | Hex value of variable |

Format specifiers: `x` hex, `d` decimal, `o` octal, `t` binary, `i` instruction, `s` string, `c` char

### Examining Variables

| Command | Description |
|---------|-------------|
| `print var` | Print variable |
| `print *ptr` | Dereference pointer |
| `print sizeof(struct)` | Size of type |
| `print array[0]@10` | Print 10 array elements |
| `print/x var` | Hex format |
| `print/t var` | Binary format |
| `ptype var` | Show variable type |

### Stack Inspection

| Command | Description |
|---------|-------------|
| `backtrace` | Show call stack |
| `backtrace full` | Stack with local variables |
| `info frame` | Current frame details |
| `info args` | Function arguments |
| `info locals` | Local variables |
| `frame 3` | Switch to frame #3 |

### Disassembly

| Command | Description |
|---------|-------------|
| `disassemble main` | Disassemble function |
| `disassemble /m main` | Mixed source + assembly |
| `disassemble /r main` | Raw bytes + assembly |
| `disassemble 0x401000,0x401100` | Address range |

### Signals

| Command | Description |
|---------|-------------|
| `info signals` | Show signal handling |
| `handle SIGSEGV nostop` | Don't stop on SIGSEGV |
| `handle SIGSEGV stop` | Stop on SIGSEGV |

### Threads

| Command | Description |
|---------|-------------|
| `info threads` | List all threads |
| `thread 3` | Switch to thread #3 |
| `thread apply all bt` | Backtrace all threads |
| `set scheduler-locking on` | Lock scheduler |

### Source Code

| Command | Description |
|---------|-------------|
| `list` | Show source code |
| `list 20,30` | Lines 20-30 |
| `info source` | Source file info |
| `info sources` | All source files |

### TUI Mode

| Key | Action |
|-----|--------|
| `Ctrl+x, a` | Toggle TUI |
| `Ctrl+x, 2` | Split window |
| `Ctrl+x, 1` | Single window |
| `Ctrl+l` | Refresh screen |

### Remote Debugging

```bash
# Target machine
gdbserver 0.0.0.0:1234 /path/to/binary

# Analysis machine
gdb -q /path/to/binary
(gdb) target remote 192.168.1.100:1234
```

### Core Dump Analysis

```bash
# Generate core dump
gcore $(pidof program)

# Analyze
gdb /path/to/binary core.PID

# Enable core dumps
ulimit -c unlimited
```

### Python API

```python
import gdb

# Get current frame
frame = gdb.selected_frame()

# Read register
rax = int(gdb.parse_and_eval("$rax"))

# Read memory
mem = gdb.selected_inferior().read_memory(address, length)

# Execute command
output = gdb.execute("info registers", to_string=True)
```

### Configuration

```bash
# ~/.gdbinit
set disassembly-flavor intel
set pagination off
set confirm off
set history save on
set history size 10000
```

### Common Patterns

```bash
# Find overflow offset
(gdb) pattern create 200
(gdb) run <<< $(pattern 200)
(gdb) pattern search $eip

# Check binary protections
(gdb) info file
(gdb) maintenance info sections

# Find /bin/sh in memory
(gdb) find &__libc_start_main,+9999999,"/bin/sh"
```
