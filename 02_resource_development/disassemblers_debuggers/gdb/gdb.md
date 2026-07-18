---
tool_name: "gdb"
display_name: "GNU Debugger (GDB)"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install gdb gdb-multiarch gdbserver"
version: "17.2"
github_repo: "https://sourceware.org/git/binutils-gdb.git"
kali_tools_page: "https://www.kali.org/tools/gdb/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: false
tags: ["reverse-engineering", "debugger", "linux", "exploit-development", "ctf"]
related_tools: ["gef", "pwndbg", "radare2", "edb-debugger", "gdbserver"]
---

# GDB - GNU Debugger

## Introduction & Overview

GDB (GNU Debugger) is the standard source-level debugger for Linux and the most widely used debugger in the world. Created by Richard Stallman in 1986 as part of the GNU Project, GDB debugs C, C++, D, Objective-C, Fortran, Java, Go, Ada, Pascal, assembly, and Modula-2 programs. It attaches to running processes, loads core dumps, and controls program execution with granular precision — setting breakpoints, stepping through instructions, inspecting memory, and evaluating expressions in real time.

GDB is not glamorous. It does not have a polished GUI, and its text interface intimidates newcomers. But it is the most powerful, most documented, and most extensible debugger ever built. Every other debugger on Linux either wraps GDB, talks to GDB's remote protocol, or competes with it from a position of inferior capability.

### What GDB Actually Does

GDB controls the execution of a target program through the ptrace system call. It can:

- Start a program and stop it at any point
- Stop a running program at a specified instruction
- Step through execution one instruction at a time
- Inspect and modify registers, memory, and variables
- Detect and analyze crashes (core dumps)
- Debug remote targets over serial, TCP, or USB
- Script automated analysis with Python, Guile, or GDB's own command language

### When to Use GDB

- Debugging any compiled program on Linux
- Exploit development (especially with GEF or Pwndbg extensions)
- Reverse engineering ELF binaries at runtime
- Analyzing crashes via core dumps
- Remote debugging of embedded devices, VMs, or containers
- Automated binary analysis with GDB Python scripting

### When NOT to Use GDB

- GUI-based analysis (use edb-debugger or Cutter)
- When a simpler tool suffices (strace, ltrace for syscall/library tracing)
- Windows debugging (use WinDbg or x64dbg — GDB on Windows is painful)
- When you need a decompiler (use Ghidra)

### Legal and Ethical Considerations

GDB is a debugging tool — completely legal to own, install, and use. Debugging software you own or have authorization to test is legal. Using GDB to debug unauthorized processes, bypass software protections, or analyze proprietary software without permission may violate the DMCA, CFAA, or equivalent local laws. GDB is the standard tool in CTF competitions, authorized penetration testing, and security research.

### Key Concepts

- **Breakpoints**: Code locations where execution pauses. GDB supports software (int3) and hardware (debug register) breakpoints.
- **Watchpoints**: Breakpoints that trigger when a memory location is read or written — ideal for finding when a variable is corrupted.
- **Catchpoints**: Breakpoints that trigger on events like library loads, exceptions, or syscall entry/exit.
- **Threads**: GDB can debug multi-threaded programs, switching between threads and inspecting each thread's state independently.
- **Inferior**: The program being debugged. GDB supports debugging multiple inferiors simultaneously.
- **Expressions**: GDB's expression language supports C-like syntax with the ability to call functions, dereference pointers, and cast types.
- **Python API**: GDB embeds a Python interpreter for writing custom commands, pretty-printers, and automation scripts.

## Installation & Setup

### System Requirements

- Linux (any architecture: x86, x86-64, ARM, ARM64, MIPS, PowerPC, RISC-V)
- Python 3.10+ for scripting support
- Terminal with readline support

### Installation via APT

```bash
sudo apt update && sudo apt install gdb gdb-multiarch gdbserver
```

**Explanation:** Installs the standard GDB, multi-architecture GDB for cross-debugging, and gdbserver for remote debugging.

Verify installation:

```bash
gdb --version
GNU gdb (GDB) 17.2
```

### Installation of GDB Multiarch

```bash
sudo apt install gdb-multiarch
```

**Explanation:** `gdb-multiarch` supports debugging binaries for architectures different from the host (e.g., debugging ARM binaries on x86-64).

### Key Packages

| Package | Purpose |
|---|---|
| `gdb` | Standard GDB for native architecture |
| `gdb-multiarch` | Multi-architecture support (ARM, MIPS, PPC, etc.) |
| `gdbserver` | Remote debugging server for target machines |
| `gdb-minimal` | Minimal GDB without Python or source support |

### Configuration

Create or edit `~/.gdbinit`:

```bash
# ~/.gdbinit
set disassembly-flavor intel
set pagination off
set confirm off
set history save on
set history filename ~/.gdb_history
set history size 10000
```

**Explanation:** These settings configure GDB with Intel assembly syntax, disable pagination for scripting, and enable command history persistence.

### Install GDB Extensions

```bash
# GEF (recommended)
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"

# Or Pwndbg
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh
```

**Explanation:** GEF and Pwndbg are the two most popular GDB extensions. Both add context displays, memory inspection, and exploit development commands.

## Basic Usage

### Starting a Session

```bash
gdb /path/to/binary
```

**Explanation:** Opens the binary in GDB, loading symbols and presenting the `(gdb)` prompt.

### Running a Program

```
(gdb) run
(gdb) run arg1 arg2
(gdb) run < input.txt
```

**Explanation:** `run` starts the program. Arguments can be passed directly, or stdin can be redirected from a file.

### Setting Breakpoints

```
(gdb) break main
(gdb) break *0x401234
(gdb) break function_name
(gdb) break file.c:42
(gdb) break main if argc == 3
```

**Explanation:** Breakpoints can be set by function name, address, source file location, or with conditions that must evaluate to true.

### Stepping Through Code

```
(gdb) stepi          # Step one instruction (into calls)
(gdb) nexti          # Step one instruction (over calls)
(gdb) step           # Step one source line (into functions)
(gdb) next           # Step one source line (over functions)
(gdb) finish         # Run until current function returns
(gdb) continue       # Continue execution to next breakpoint
```

**Explanation:** These commands control execution granularity — from single CPU instructions to source-level lines.

### Inspecting Registers

```
(gdb) info registers
(gdb) info registers eax ebx ecx
(gdb) print $eax
(gdb) set $eax = 0x0
```

**Explanation:** Display or modify individual registers. The `$` prefix identifies register names in expressions.

### Examining Memory

```
(gdb) x/10x $rsp              # 10 hex words from RSP
(gdb) x/20i $rip              # 20 instructions from RIP
(gdb) x/s 0x402000            # String at address
(gdb) x/40wx $rsp             # 40 hex words (detailed stack dump)
(gdb) x/10i main              # 10 instructions starting at main
```

**Explanation:** The `x` (examine) command displays memory in various formats: hex, decimal, octal, instructions, strings.

### Examining Variables

```
(gdb) print variable_name
(gdb) print sizeof(struct_name)
(gdb) print *pointer
(gdb) print array[0]@10       # Print 10 elements starting at array[0]
(gdb) print/x variable        # Print in hexadecimal
```

**Explanation:** GDB evaluates C-like expressions including variable access, pointer dereferencing, type casting, and array slicing.

### Inspecting the Stack

```
(gdb) backtrace
(gdb) backtrace full
(gdb) info frame
(gdb) info args
(gdb) info locals
```

**Explanation:** `backtrace` shows the call stack. `full` includes local variables. `info frame` shows the current stack frame details.

### Disassembly

```
(gdb) disassemble main
(gdb) disassemble /m main      # Mixed source and assembly
(gdb) disassemble /r main      # Raw bytes with assembly
(gdb) disassemble 0x401000,0x401100  # Address range
```

**Explanation:** GDB uses Capstone internally for disassembly. The `/m` flag interleaves source code with assembly when debug symbols are available.

## Intermediate Usage

### Watchpoints

```
(gdb) watch variable          # Break when variable is written
(gdb) rwatch variable         # Break when variable is read
(gdb) awatch variable         # Break on any access (read or write)
```

**Explanation:** Watchpoints are hardware-assisted breakpoints that trigger when memory content changes — essential for finding which code corrupts a variable.

### Core Dump Analysis

```bash
# Generate a core dump
gcore $(pidof target_program)

# Analyze with GDB
gdb /path/to/binary core.12345

# Or enable core dumps
ulimit -c unlimited
echo "/tmp/core.%e.%p" > /proc/sys/kernel/core_pattern
```

**Explanation:** Core dumps capture the state of a crashed process for post-mortem analysis. GDB loads core dumps to show the exact state at crash time.

### Remote Debugging

```bash
# On target machine
gdbserver 0.0.0.0:1234 /path/to/binary

# On analysis machine
gdb /path/to/binary
(gdb) target remote 192.168.1.100:1234
```

**Explanation:** gdbserver runs the binary on the target machine, accepting GDB connections over TCP. The analysis machine connects remotely for debugging.

### GDB TUI (Text User Interface)

```bash
gdb -tui /path/to/binary
```

**Explanation:** Launches GDB in TUI mode, showing a split screen with the source code/assembly and the GDB command prompt.

| Key | Function |
|---|---|
| Ctrl+x, a | Toggle TUI mode |
| Ctrl+x, 2 | Split window (source + assembly) |
| Ctrl+x, 1 | Single window |
| Ctrl+l | Refresh screen |
| Up/Down | Scroll through source |

### Python Scripting

```python
# In GDB prompt:
(gdb) python
> import gdb
> for frame in gdb.selected_inferior().frames():
>     print(frame.name())
> end
```

**Explanation:** GDB embeds a Python interpreter for writing custom analysis commands, pretty-printers, and automation scripts.

### Custom GDB Commands

```python
import gdb

class FindVuln(gdb.Command):
    """Find calls to dangerous functions"""
    def __init__(self):
        super().__init__("find-vuln", gdb.COMMAND_USER)

    def invoke(self, arg, from_tty):
        dangerous = ["gets", "strcpy", "sprintf", "scanf"]
        result = gdb.execute("info functions", to_string=True)
        for func in dangerous:
            if func in result:
                print(f"[!] Found: {func}")

FindVuln()
```

**Explanation:** Custom GDB commands are Python classes that register with GDB's command infrastructure and become available at the `(gdb)` prompt.

### Pretty-Printing Structures

```python
class MyStructPrinter:
    def __init__(self, val):
        self.val = val

    def to_string(self):
        return f"MyStruct(a={self.val['a']}, b={self.val['b']}, c={self.val['c']})"

def lookup_type(val):
    if str(val.type) == "MyStruct":
        return MyStructPrinter(val)
    return None

gdb.pretty_printers.append(lookup_type)
```

**Explanation:** Pretty-printers customize how GDB displays complex data structures, making struct inspection readable instead of raw hex dumps.

### Catching Syscalls

```
(gdb) catch syscall write
(gdb) catch syscall open
(gdb) catch syscall execve
```

**Explanation:** Catchpoints trigger on specific system calls, useful for monitoring file operations, network activity, or process creation.

### Information Commands Reference

| Command | Purpose |
|---|---|
| `info breakpoints` | List all breakpoints |
| `info threads` | List all threads |
| `info sharedlibrary` | List loaded shared libraries |
| `info proc mappings` | Memory map (like vmmap in GEF) |
| `info signals` | Signal handling information |
| `info variables` | All global and static variables |
| `info types` | All type names |

## Advanced Usage

### Multi-Threaded Debugging

```
(gdb) info threads
(gdb) thread 3
(gdb) thread apply all backtrace
(gdb) set scheduler-locking on
```

**Explanation:** GDB can debug multi-threaded programs, switching between threads, applying commands to all threads, and locking the scheduler to prevent thread interleaving during debugging.

### Follow-Child Mode (Fork Debugging)

```
(gdb) set follow-fork-mode child
(gdb) set detach-on-fork off
```

**Explanation:** When a program forks, GDB can follow either the parent or child process. `detach-on-fork off` keeps both inferiors connected.

### Process Record and Replay

```
(gdb) target record-full
(gdb) reverse-stepi
(gdb) reverse-continue
```

**Explanation:** Records the execution of a program, then allows stepping backward through instructions — invaluable for understanding how a crash occurred.

### Scripting with GDB Commands

```bash
# batch.gdb — Run GDB commands non-interactively
set pagination off
file /path/to/binary
break main
run
disassemble main
info registers
quit
```

```bash
gdb -batch -x batch.gdb /path/to/binary
```

**Explanation:** GDB's batch mode executes commands without interactive input, enabling automated analysis pipelines.

### Analyzing Stripped Binaries

```
(gdb) info functions
# If no symbols, use pattern matching:

(gdb) disassemble $pc, +50
(gdb) x/10i $pc
(gdb) info proc mappings
# Identify functions by PLT entries:
(gdb) disassemble printf
```

**Explanation:** Without debug symbols, analysis relies on PLT entries, library function addresses, and instruction pattern recognition.

### Kernel Debugging with QEMU

```bash
# Start QEMU
qemu-system-x86_64 -s -S -kernel bzImage -append "nokaslr"

# Connect GDB
gdb vmlinux
(gdb) target remote :1234
(gdb) hbreak *start_kernel
(gdb) continue
```

**Explanation:** GDB connects to QEMU's GDB stub for kernel debugging. Hardware breakpoints (hbreak) are used because kernel text may be read-only.

### Debugging with Source Code

```bash
# Install debug symbols
sudo apt install libc6-dbg

# Compile with debug info
gcc -g -O0 -o target target.c

# Debug with source
gdb -q target
(gdb) list main
(gdb) break target.c:25
(gdb) info source
```

**Explanation:** Debug symbols (`-g` flag) enable source-level debugging with line numbers, variable names, and type information.

### Conditional Breakpoint Patterns

```
# Break after a function has been called 10 times
(gdb) break function_name
(gdb) ignore 1 9

# Break when a pointer equals a specific value
(gdb) break *0x401234 if *(int*)0x601040 == 0xdeadbeef

# Break in a loop iteration
(gdb) break loop_body if i == 999
```

**Explanation:** Conditional breakpoints reduce noise by only stopping execution when specific conditions are met, crucial for debugging loops and high-frequency code paths.

### Memory Corruption Detection

```
(gdb) watch -l *(char*)0x601080
(gdb) commands
  > silent
  > printf "Corrupted at %p, backtrace:\n", $rip
  > backtrace
  > continue
  > end
```

**Explanation:** This setup watches a memory location and automatically prints a backtrace when it's modified — catching the exact moment corruption occurs.

## Real-World Scenarios

### Scenario 1: CTF Binary Exploitation

```bash
# Load the binary
gdb -q ./vuln_binary

# Quick checks
(gdb) info file
(gdb) disassemble main

# Find the overflow
(gdb) run AAAA
# Observe crash with 0x41414141 in EIP/RIP

# Develop the exploit
(gdb) pattern create 200
(gdb) run <<< $(pattern 200)
(gdb) pattern search $eip

# Build payload with the offset
```

**Explanation:** GDB's step-by-step debugging lets you trace the exact behavior of the vulnerability, from input to crash, with full visibility of every register and memory change.

### Scenario 2: Malware Analysis

```bash
gdb -q malware.elf
(gdb) set disassembly-flavor intel
(gdb) disassemble main
# Identify the suspicious function
(gdb) break *suspicious_func
(gdb) run
(gdb) x/s $rdi          # Check first argument (string?)
(gdb) x/10x $rsi        # Check second argument
(gdb) nexti
# Follow the execution to understand behavior
```

**Explanation:** GDB steps through malware execution at the instruction level, revealing system calls, network activity, and file operations in real time.

### Scenario 3: Debugging a Segfault

```bash
gdb -q crashing_program
(gdb) run
# Program crashes

(gdb) backtrace
#0  0x00007ffff7e8a123 in ?? () from /lib/libc.so.6
#1  0x00005555555551a3 in process_data (buf=0x0) at main.c:42

(gdb) frame 1
(gdb) info locals
(gdb) print buf
# buf = 0x0 (NULL pointer — that's the bug)

(gdb) list 38,45
# View the source code around the crash
```

**Explanation:** GDB's backtrace and frame inspection immediately identify the crash location and reveal that a NULL pointer dereference is the root cause.

### Scenario 4: Remote Debugging an Embedded Device

```bash
# On the embedded target
gdbserver 0.0.0.0:1234 /usr/bin/target_app

# On the analysis workstation
gdb-multiarch -q target_app
(gdb) set architecture arm
(gdb) target remote 192.168.1.50:1234
(gdb) break main
(gdb) continue
```

**Explanation:** GDB's remote debugging and multi-architecture support make it possible to debug embedded devices (ARM, MIPS, PowerPC) from a standard x86-64 workstation.

### Scenario 5: Analyzing a Race Condition

```
(gdb) set scheduler-locking on
(gdb) break vulnerable_function
(gdb) thread apply all break shared_resource
(gdb) continue
# Examine lock state and thread interactions
(gdb) info threads
(gdb) thread 2
(gdb) print lock_variable
```

**Explanation:** GDB's threading support with scheduler locking enables controlled reproduction of race conditions by controlling exactly when each thread executes.

### Scenario 6: Analyzing a Heap Corruption

A program crashes intermittently with heap corruption:

```bash
gdb -q /path/to/program
(gdb) set environment MALLOC_CHECK_=3
(gdb) run
# When malloc detects corruption, it aborts

(gdb) backtrace full
# Identify the corrupted chunk

(gdb) watch -l *(char*)chunk_address
(gdb) commands
  > silent
  > printf "Heap write at $rip\n"
  > backtrace
  > continue
  > end
```

**Explanation:** GDB's watchpoints combined with MALLOC_CHECK_ catch heap corruption at the exact moment it occurs, revealing the offending write.

### Scenario 7: Analyzing Shared Library Behavior

```bash
gdb -q /usr/bin/python3
(gdb) break Py_Initialize
(gdb) run
(gdb) info sharedlibrary
# See which libraries are loaded

(gdb) break dlopen
(gdb) continue
(gdb) print (char*)$rdi
# See which library is being loaded dynamically
```

**Explanation:** GDB can break on shared library loading events and inspect dynamic linker behavior — critical for understanding plugin architectures and LD_PRELOAD attacks.

### Scenario 8: Debugging a Kernel Module with QEMU

```bash
# Start QEMU with debug kernel
qemu-system-x86_64 -s -S \
    -kernel /boot/vmlinuz-$(uname -r) \
    -initrd /boot/initrd.img-$(uname -r) \
    -append "nokaslr console=ttyS0" \
    -nographic -monitor none

# In another terminal, connect GDB
gdb vmlinux
(gdb) set architecture i386:x86-64
(gdb) target remote :1234
(gdb) hbreak start_kernel
(gdb) continue
# Breaks at start_kernel
(gdb) break module_init_function
(gdb) continue
# Your module's init function is now debuggable
```

**Explanation:** Kernel module debugging with QEMU and GDB provides full visibility into kernel execution without risking system stability. Hardware breakpoints are required because kernel text is read-only.

## Detection & Defense

### How Defenders Detect GDB

- **ptrace**: GDB attaches via `ptrace(PTRACE_ATTACH)`, which is detectable
- **/proc/self/status**: The TracerPid field becomes non-zero when debugged
- **Timing checks**: Single-stepping dramatically slows execution
- **Signal behavior**: GDB intercepts signals before the target sees them
- **Process enumeration**: `ps aux | grep gdb` reveals the debugger
- **Anti-debugging APIs**: `ptrace(PTRACE_TRACEME)` fails when a debugger is attached

### Blue Team Response

- Monitor for ptrace system calls from unexpected processes
- Implement anti-debugging checks in critical applications
- Alert on gdb/gdbserver process creation on production systems
- Check for GDB installations and `.gdbinit` files on compromised systems
- Monitor network connections on port 1234 (default gdbserver port)

## Troubleshooting

### Common Errors and Solutions

**Error: `No symbol table is loaded`**

```bash
gcc -g -o target target.c
gdb -q target
```

**Explanation:** The binary was compiled without debug symbols. Recompile with `-g` or install debug packages.

**Error: `Cannot access memory at address 0x0`**

This is a NULL pointer dereference — the program is trying to access address zero.

```
(gdb) backtrace
```

**Explanation:** Use the backtrace to find which function is dereferencing NULL, then check the calling code for missing null checks.

**Error: `Remote communication error`**

```
(gdb) target remote 192.168.1.100:1234
# Connection refused
```

Verify gdbserver is running on the target and the port is open:

```bash
# On target
gdbserver 0.0.0.0:1234 /path/to/binary
```

**Error: `The program is not being run`**

```
(gdb) continue
# The program hasn't been started yet
(gdb) run
```

**Explanation:** You must use `run` to start the program before `continue` works. `continue` only resumes an already-running program.

**Error: `Warning: Cannot insert breakpoint`**

Hardware breakpoints require CPU debug registers (4 maximum on x86). Use software breakpoints or reduce the number of active hardware breakpoints.

### Performance Optimization

- Use `set pagination off` for scripted/automated sessions
- Disable source line display with `set disassembly-flavor intel` instead of `/m` flag
- For core dump analysis, load with `gdb -q binary core` (quiet mode)
- Use `set confirm off` to skip confirmation prompts

### FAQ

**Q: GDB vs LLDB?**
A: GDB is the standard on Linux with better documentation and community support. LLDB (from LLVM) has faster startup and better expression evaluation. For Linux RE/exploit dev, GDB with GEF is the superior choice.

**Q: How do I debug a setuid program?**
A: `sudo gdb /usr/bin/su` then `run`. GDB needs root to follow setuid processes.

**Q: Can GDB debug Android apps?**
A: Yes, via `gdbserver` running on the Android device and `gdb-multiarch` on the host. Alternatively, use Android Studio's built-in debugger.

## Resources

### Official Documentation
- [GDB Manual](https://sourceware.org/gdb/documentation/)
- [GDB Python API](https://sourceware.org/gdb/current/onlinedocs/gdb/Python-API.html)
- [GDB Remote Serial Protocol](https://sourceware.org/gdb/current/onlinedocs/gdb/Remote-Protocol.html)

### Tutorials and Guides
- [GDB Tutorial by THEGDB](https://www.yourwebsite.com/gdb-tutorial)
- [GDB Pocket Reference](https://www.oreilly.com/library/view/gdb-pocket-reference/9780596517708/)
- [Advanced GDB Usage](https://undos.com/blog/advanced-gdb-usage/)

### Books
- *The Art of Debugging with GDB, DDD, and Eclipse* by Norman Matloff
- *GDB Pocket Reference* by Arnold Robbins
- *Practical Binary Analysis* by Dennis Andriesse

### Related Tools
- [GEF](../gef/gef.md) — GDB extension for exploit development
- [Pwndbg](https://github.com/pwndbg/pwndbg) — Alternative GDB enhancement
- [edb-debugger](../edb-debugger/edb-debugger.md) — GUI debugger for Linux
- [Radare2](../radare2/radare2.md) — Command-line debugger with disassembly
- [Ghidra](../ghidra/ghidra.md) — Reverse engineering suite with debugger

## GDB Quick Reference Table

| Category | Command | Description |
|----------|---------|-------------|
| **Execution** | `run` | Start program |
| | `continue` | Continue to breakpoint |
| | `stepi` | Step one instruction (into) |
| | `nexti` | Step one instruction (over) |
| | `finish` | Run until function returns |
| | `kill` | Kill the inferior process |
| **Breakpoints** | `break main` | Break at function |
| | `break *0x401234` | Break at address |
| | `break file.c:42` | Break at source line |
| | `break func if x>10` | Conditional breakpoint |
| | `watch var` | Break on write |
| | `rwatch var` | Break on read |
| | `delete N` | Delete breakpoint N |
| | `disable N` | Disable breakpoint N |
| **Registers** | `info registers` | Show all registers |
| | `print $rax` | Print register |
| | `set $rax = 0` | Modify register |
| **Memory** | `x/Nfx ADDR` | Examine N items |
| | `x/Nix ADDR` | Examine N instructions |
| | `x/s ADDR` | Examine string |
| **Stack** | `backtrace` | Show call stack |
| | `info frame` | Frame details |
| | `info args` | Function arguments |
| | `info locals` | Local variables |
| | `frame N` | Switch frame |
| **Source** | `list` | Show source |
| | `info source` | Source info |
| **Threads** | `info threads` | List threads |
| | `thread N` | Switch thread |
| | `thread apply all bt` | Backtrace all |
| **Info** | `info functions` | List functions |
| | `info breakpoints` | List breakpoints |
| | `info sharedlibrary` | Loaded libraries |
| | `info proc mappings` | Memory map |
| **Config** | `set disassembly-flavor intel` | Intel syntax |
| | `set pagination off` | Disable paging |
| | `set confirm off` | Skip confirmations |
