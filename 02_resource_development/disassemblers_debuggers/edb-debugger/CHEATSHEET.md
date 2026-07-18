# EDB (Evan's Debugger) Cheatsheet

## Quick Reference Card

### Installation

```bash
sudo apt install edb-debugger edb-debugger-plugins
```

### Launch

```bash
edb                                      # Start GUI
edb --run /path/to/binary                # Open binary directly
edb --attach 1234                        # Attach to PID
edb --symbols /lib/libc.so.6 > libc.map  # Generate symbol map
```

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `F2` | Toggle breakpoint at cursor |
| `F7` | Step into (execute one instruction) |
| `F8` | Step over (execute one instruction, skip calls) |
| `F9` | Run / Continue to breakpoint |
| `F5` | Run to cursor |
| `Ctrl+F2` | Restart debugging session |
| `Alt+F4` | Close EDB |

### Interface Panels

| Panel | Purpose |
|-------|---------|
| CPU/Disassembly | Disassembled instructions at current EIP/RIP |
| Registers | General purpose, flags, segments |
| Stack | Current stack contents |
| Dump/Hexdump | Memory dump at any address |
| Log | Events, breakpoints, messages |
| Breakpoints | Manage all active breakpoints |
| Bookmarks | Saved addresses for quick navigation |

### Essential Commands

| Action | How |
|--------|-----|
| Set breakpoint | Click line → `F2` |
| Step into | `F7` |
| Step over | `F8` |
| Run to cursor | Click target → `F5` |
| Run | `F9` |
| Follow in dump | Right-click → Follow in Dump |
| Assemble | Right-click → Assemble |
| Patch binary | File → Save after patching |

### Register View

| Register | Purpose |
|----------|---------|
| RAX/EAX | Return value / accumulator |
| RBX/EBX | Base register |
| RCX/ECX | Counter register |
| RDX/EDX | Data register |
| RSI/ESI | Source index (string ops) |
| RDI/EDI | Destination index (string ops) |
| RSP/ESP | Stack pointer |
| RBP/EBP | Base pointer (frame) |
| RIP/EIP | Instruction pointer |
| RFLAGS | CPU flags |

### Memory Examination

```
# Navigate memory:
Right-click address → Follow in Dump
Dump panel → enter address in address bar

# Memory regions:
vmmap equivalent: View → Memory Map
```

### Conditional Breakpoints

```
# Right-click breakpoint → Properties → Condition
# Examples:
EAX == 0x42
[ESP+4] == 0x12345678
*(int*)0x601040 > 100
```

### Hardware Breakpoints

```
# Breakpoints panel → Set Type → Hardware
# Options: Execute, Read, Write
# Max 4 per thread (x86 DR0-DR3)
```

### Symbol Maps

```bash
# Generate symbol maps for stripped binaries
edb --symbols /lib/x86_64-linux-gnu/libc.so.6 > libc.map

# Generate maps for all libs
for i in $(ls /lib); do
    edb --symbols $i > $(basename $i).map
done

# Place in ~/.edb/symbols/
```

### Plugin Commands

| Plugin | Purpose |
|--------|---------|
| Heap Analysis | Inspect heap allocations |
| ROP Gadgets | Find ROP gadgets for exploitation |
| Breakpoint Manager | Advanced breakpoint management |
| Call Stack | Full call stack view |

### Patching

```
1. Navigate to target instruction
2. Right-click → Assemble
3. Enter new instruction (e.g., "nop")
4. Changes take effect immediately
5. File → Save to write to disk
```

### Common Workflows

#### Exploit Development

```
1. Load vulnerable binary
2. Break on vulnerable function
3. Run with oversized input
4. Inspect stack at crash
5. Identify EIP overwrite offset
6. Find ROP gadgets
7. Craft payload
```

#### Malware Analysis

```
1. Load suspicious binary
2. Break on network APIs (connect, send)
3. Run and inspect arguments
4. Step through execution
5. Document behavior
```

### Tips

- Use software breakpoints (F2) by default; hardware only when code is read-only
- Right-click register → Follow in Dump to view memory at register address
- Check `ptrace` protection: if breakpoints fail, the binary may anti-debug
- Generate symbol maps for stripped binaries before analysis
