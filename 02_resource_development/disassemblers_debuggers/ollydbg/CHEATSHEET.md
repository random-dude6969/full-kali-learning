# OllyDbg Cheatsheet

## Quick Reference Card

### Installation & Launch

```bash
# Install Wine support
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt install wine32 ollydbg

# Launch
wine /usr/share/ollydbg/OLLYDBG.EXE

# Open specific binary
wine /usr/share/ollydbg/OLLYDBG.EXE target.exe
```

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `F2` | Set/clear breakpoint |
| `F7` | Step into (execute one instruction) |
| `F8` | Step over (execute one instruction, skip calls) |
| `F9` | Run until breakpoint/exception |
| `F12` | Pause execution |
| `Ctrl+F2` | Restart debugging session |
| `Alt+F4` | Close OllyDbg |
| `Space` | Assemble instruction at cursor |
| `Ctrl+G` | Go to address |
| `Ctrl+F` | Search in disassembly |

### Interface Panels

| Panel | Purpose |
|-------|---------|
| CPU (Disassembly) | Disassembled code with annotations |
| Registers | EAX-EDI, EIP, flags, segments |
| Hex Dump | Memory dump view |
| Stack | Current stack contents |
| Log | Events, breakpoints, messages |

### Register Reference

| Register | Purpose |
|----------|---------|
| EAX | Return value / accumulator |
| EBX | Base register |
| ECX | Counter register |
| EDX | Data register |
| ESI | Source index (string ops) |
| EDI | Destination index (string ops) |
| EBP | Base pointer (stack frame) |
| ESP | Stack pointer |
| EIP | Instruction pointer |
| EFLAGS | CPU flags (ZF, CF, SF, OF, etc.) |

### Setting Breakpoints

| Method | How |
|--------|-----|
| Software breakpoint | Click line → `F2` |
| Conditional breakpoint | Right-click BP → Condition |
| Memory breakpoint | Debug → Memory Breakpoints |
| API breakpoint | Search → All intermodular calls |

### Conditional Breakpoint Expressions

```
[ESP+4] == 0x12345678         # Check stack argument
EAX == 0                      # Check register value
[EBP-8] > 100                 # Check local variable
```

### Searching

| Search | How |
|--------|-----|
| Text strings | Right-click → Search for → All referenced text strings |
| Intermodular calls | Right-click → Search for → All intermodular calls |
| Binary sequence | Right-click → Search for → Binary sequence |
| Commands | Right-click → Search for → Command |

### Patching

```
1. Navigate to instruction
2. Press Space → Assemble dialog appears
3. Enter new instruction (e.g., "NOP")
4. Click Assemble
5. Right-click → Copy to executable → Save file
```

### Analysis Workflow

```
1. Open binary (File → Open)
2. Auto-analysis runs on load
3. Check imports (View → Executable modules → right-click → View import table)
4. Find string references (Search for → All referenced text strings)
5. Set breakpoints on interesting functions
6. Run (F9) and analyze behavior
7. Patch if needed
8. Save (Copy to executable → Save file)
```

### Unpacking Workflow

```
1. Load packed binary
2. Find OEP (Original Entry Point) using:
   - Trace past unpacking stub
   - Break on VirtualProtect/WriteProcessMemory
3. At OEP: Dump process with OllyDump plugin
4. Fix PE headers with PE header fixer
5. Analyze unpacked binary
```

### Anti-Debugging Bypass

| Check | Bypass |
|-------|--------|
| IsDebuggerPresent | Patch PEB.BeingDebugged |
| NtQueryInformationProcess | Use HideOD plugin |
| Timing (RDTSC) | Patch timing comparison |
| Hardware breakpoint detection | Use software breakpoints |
| Window class check | Hide OllyDbg window |

### Plugins

| Plugin | Purpose |
|--------|---------|
| OllyDump | Dump process memory |
| HideOD | Hide debugger from detection |
| Phantom | Anti-anti-debugging |
| ODBGScript | Scripting support |

### Tips

- OllyDbg is 32-bit only — use x64dbg for 64-bit binaries
- Always check the log window for analysis messages
- Use conditional breakpoints to avoid noise from loops
- Generate symbol maps for stripped binaries before analysis
- Save analysis notes in the log window
