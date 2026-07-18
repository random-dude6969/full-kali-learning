---
tool_name: "edb-debugger"
display_name: "Evan's Debugger"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install edb-debugger"
version: "1.3.0"
github_repo: "https://github.com/eteran/edb-debugger"
kali_tools_page: "https://www.kali.org/tools/edb-debugger/"
difficulty_level: "intermediate"
requires_root: false
gui_available: true
tags: ["reverse-engineering", "debugger", "linux", "x86", "x86-64", "gui"]
related_tools: ["ollydbg", "gdb", "gef", "radare2"]
---

# EDB - Evan's Debugger

## Introduction & Overview

EDB (Evan's Debugger) is a cross-platform graphical debugger for x86, x86-64, and AArch32 binaries. Built as a Linux-native spiritual successor to OllyDbg, EDB combines a clean Qt-based interface with the Capstone disassembly engine to deliver an intuitive debugging experience on Linux. Created by Evan Teran, EDB fills the gap that GDB leaves in the GUI space — providing the visual register views, memory dumps, and breakpoint management that OllyDbg users expect, but natively on Linux.

### What EDB Actually Does

EDB attaches to Linux processes (or loads ELF executables directly) and provides a multi-pane debugging environment: CPU disassembly, register state, memory dumps, stack view, and a log window. It uses ptrace for process control and Capstone for instruction disassembly. EDB supports hardware and software breakpoints, memory patching, function tracing, and a plugin system for extending its capabilities.

### When to Use EDB

- Debugging ELF binaries on Linux with a visual interface
- When GDB's text interface feels too slow for quick analysis
- Exploit development on Linux where register and stack visibility matters
- Quick binary patching and NOP-ing instructions in running processes
- Analyzing x86/x64 shellcode step by step

### When NOT to Use EDB

- Remote debugging (GDB with gdbserver is better)
- Kernel debugging (use KGDB or QEMU)
- Multi-process or multi-threaded debugging at scale (GDB handles this better)
- When you need scripting automation (GDB + GEF/Python is superior)

### Legal and Ethical Considerations

EDB is a debugging tool that interacts with live processes. Attaching to processes you do not own requires root privileges and may violate system policies. Authorized penetration testing, security research, and educational use are the intended purposes. Debugging production systems without authorization is illegal in most jurisdictions.

### Key Concepts

- **ptrace**: Linux system call that allows one process to control another — EDB's core mechanism.
- **Software Breakpoints**: Int3 (0xCC) instructions inserted into the target's code.
- **Hardware Breakpoints**: CPU debug registers (DR0-DR3) for breaking on data access without modifying code.
- **Capstone Disassembly**: EDB uses Capstone for instruction decoding, supporting x86, x86-64, and ARM.
- **Plugins**: EDB supports plugins for additional analysis features like heap analysis and ROP gadget finding.

## Installation & Setup

### System Requirements

- Linux kernel 3.0+
- Qt 5.15+ or Qt 6.x
- Capstone >= 3.0
- Graphviz >= 2.38.0 (optional, for call graphs)
- X11 display server

### Installation via APT

```bash
sudo apt update && sudo apt install edb-debugger edb-debugger-plugins
```

**Explanation:** Installs EDB and its companion plugins package, which includes heap analysis, ROP gadget finding, and other extensions.

### Launch EDB

```bash
edb
```

**Explanation:** Starts the EDB graphical debugger. The main window displays the CPU view, register panel, and log window.

### Build from Source (Latest Version)

```bash
git clone --recursive https://github.com/eteran/edb-debugger.git
cd edb-debugger
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

**Explanation:** Builds the latest EDB from source for access to newest features and bug fixes. Requires Qt, Capstone, and a C++17 compiler.

### Verify Installation

```bash
edb --version
```

**Explanation:** Displays the EDB version string, confirming a successful installation.

## Basic Usage

### Opening a Binary

```bash
edb --run /bin/ls
```

**Explanation:** Opens the `ls` binary in EDB's debugger, pausing at the program entry point before `main()`.

### Attaching to a Running Process

```bash
edb --attach 1234
```

**Explanation:** Attaches EDB to the process with PID 1234, pausing execution for inspection.

### Key Interface Elements

| Panel | Purpose |
|---|---|
| CPU/Disassembly | Shows disassembled instructions at the current execution point |
| Registers | Displays general-purpose, segment, and flag registers |
| Stack | Shows the current stack contents |
| Dump/Hexdump | Memory dump view for any address |
| Log | Events, breakpoints, and debugger messages |

### Essential Keyboard Shortcuts

| Key | Function |
|---|---|
| F2 | Toggle breakpoint at cursor |
| F7 | Step into (execute one instruction) |
| F8 | Step over (skip over function calls) |
| F9 | Run until breakpoint or exception |
| F5 | Run to cursor |
| Ctrl+F2 | Restart debugging session |

### Generating Symbol Maps

```bash
edb --symbols /lib/libc.so.6 > libc.so.6.map
```

**Explanation:** Generates a symbol map file for a shared library, useful for identifying functions in stripped binaries.

### Setting Breakpoints

Click on a disassembly line and press F2. The line highlights in red to indicate an active breakpoint. Use the Breakpoints panel to manage all breakpoints, set conditions, or disable them temporarily.

### Examining Memory

1. In the CPU view, right-click any memory address
2. Select "Follow in Dump" to view the raw bytes
3. Use the Dump panel's address bar to navigate to any memory location

### Patching Instructions

1. Navigate to the instruction to patch in the CPU view
2. Right-click → Assemble
3. Enter the new instruction (e.g., `nop`)
4. The change takes effect immediately in the running process
5. File → Save to write changes to disk

## Intermediate Usage

### Conditional Breakpoints

Right-click a breakpoint → Properties → Condition:

```
EAX == 0x42
```

**Explanation:** The breakpoint only triggers when EAX equals 0x42, filtering out unwanted hits during loops.

### Memory Breakpoints (Hardware)

1. Set a breakpoint type to "Hardware Breakpoint" in the Breakpoints panel
2. Choose the access type: Execute, Read, or Write
3. Specify the memory address and size (1, 2, or 4 bytes)

Hardware breakpoints use CPU debug registers and do not modify the target's code — useful for watching variable modifications without interfering with anti-debugging checks.

### Function Graph View

1. Right-click in the CPU view → Graph → Follow
2. EDB renders a visual control flow graph using Graphviz
3. Nodes represent basic blocks, edges represent jumps

### Analyzing with Plugins

EDB ships with several plugins:

| Plugin | Function |
|---|---|
| Heap Analysis | Inspect heap allocations and detect overflows |
| ROP Gadgets | Find useful gadgets for ROP chain development |
| Breakpoint Manager | Advanced breakpoint management and conditions |
| Call Stack | View the full call stack with arguments |

### Trace Execution

1. Start debugging a function of interest
2. Enable tracing: Debug → Trace
3. Step through the execution
4. View the trace log to see every instruction executed with register changes

### Debugging Stripped Binaries

For binaries without debug symbols:

```bash
# Generate symbols from the binary
edb --symbols /path/to/binary > binary.map

# Place the .map file in EDB's symbols directory
cp binary.map ~/.edb/symbols/

# EDB will auto-load symbols on next open
```

**Explanation:** Symbol maps help EDB label functions in stripped binaries, making navigation significantly easier.

## Advanced Usage

### ROP Gadget Discovery

Use EDB's ROP gadget plugin to find useful instruction sequences:

1. Open the target binary
2. Plugins → ROP Gadgets
3. Filter by architecture and instruction patterns
4. Export gadgets for use in exploit development

### Exploit Development Workflow

EDB excels at exploit development on Linux:

1. Load the vulnerable binary
2. Set a breakpoint on the vulnerable function (e.g., `strcpy`)
3. Run with a crafted payload
4. At the breakpoint, inspect the stack to see the overflow
5. Step through to find the return address overwrite
6. Use the ROP gadget finder to build a chain
7. Craft the final payload

### Debugging Shared Libraries

```bash
edb --run /usr/bin/python3
```

**Explanation:** Load the main executable first, then set breakpoints on shared library functions after they're loaded via dynamic linking.

### Analyzing Anti-Debugging Techniques

Common techniques and EDB's responses:

| Technique | Bypass in EDB |
|---|---|
| ptrace self-check | Patch `ptrace` to return 0 |
| Timing checks | NOP the RDTSC comparison |
| `/proc/self/status` | Intercept file reads |
| Signal-based detection | Handle SIGTRAP gracefully |

### Multi-Architecture Debugging

EDB supports AArch32 (ARM 32-bit) debugging on compatible systems:

```bash
edb --run arm_binary
```

**Explanation:** EDB detects the target architecture automatically and adjusts the disassembly and register display accordingly.

## Real-World Scenarios

### Scenario 1: Buffer Overflow Analysis

A web application has a buffer overflow in its input handler:

1. Load the vulnerable daemon in EDB
2. Set a breakpoint on the `recv()` or `read()` call
3. Send an oversized payload from a remote machine
4. At the breakpoint, examine the stack
5. Identify the offset to EIP/RIP
6. Calculate the return address overwrite location

### Scenario 2: CTF Binary Exploitation

A CTF challenge provides a vulnerable ELF binary:

1. Load in EDB, check protections: `checksec` (via GDB or command line)
2. Find the vulnerability with the disassembly view
3. Use the ROP gadget plugin to build a chain
4. Test the exploit step-by-step in the debugger
5. Verify the payload works before submitting

### Scenario 3: Malware Dynamic Analysis

A suspicious ELF binary needs behavioral analysis:

1. Load in EDB
2. Run until the first network call (break on `connect`)
3. Inspect the arguments — IP address and port are visible in registers/stack
4. Continue to the next system call
5. Trace the execution flow to understand the malware's behavior

### Scenario 4: Kernel Module Debugging

For kernel module debugging with QEMU:

```bash
# Start QEMU with GDB stub
qemu-system-x86_64 -s -S -kernel vmlinuz -append "root=/dev/sda1"

# Connect EDB via remote debugging
edb --remote localhost:1234
```

**Explanation:** EDB can connect to QEMU's GDB stub for debugging kernel code during module development.

## Detection & Defense

### How Defenders Detect EDB

- Process name: `edb` visible in process listings
- Qt library loading: Large Qt5/Qt6 libraries loaded into memory
- Capstone library: `libcapstone.so` loaded by the debugger
- ptrace calls: The debugger makes ptrace syscalls to control the target
- File access: Debug symbols and map files accessed from `~/.edb/`

### Blue Team Response

- Monitor for debugger attachment via `/proc/<pid>/status` TracerPid field
- Alert on `ptrace` system calls from unexpected processes
- Implement anti-debugging checks in sensitive applications
- Check for EDB installation directories on compromised systems

## Troubleshooting

### Common Errors and Solutions

**Error: `edb: command not found`**

```bash
sudo apt install edb-debugger
```

**Explanation:** The EDB package must be installed separately. It is not included in the default Kali installation.

**Error: `Could not find Qt5 libraries`**

```bash
sudo apt install libqt5widgets5 libqt5svg5 libqt5xml5
```

**Explanation:** EDB depends on Qt5 libraries for its GUI. Install the required Qt5 packages.

**Error: Breakpoints not working in optimized binaries**

Optimized code may inline functions or reorder instructions, making breakpoints unreliable. Compile with `-O0 -g` for debuggable binaries.

**Error: EDB crashes on specific binaries**

Update to the latest version from source, as the Kali package may be outdated:

```bash
git clone --recursive https://github.com/eteran/edb-debugger.git
cd edb-debugger && mkdir build && cd build
cmake .. && make -j$(nproc)
```

**Explanation:** The Kali package is often several versions behind. Building from source provides the latest bug fixes.

### Performance Optimization

- Reduce the hex dump window update frequency for large memory views
- Disable unnecessary plugins to reduce startup time
- Use software breakpoints instead of hardware when possible (hardware breakpoints are limited to 4 per thread)

### FAQ

**Q: EDB vs GDB — when to use which?**
A: Use EDB for quick visual debugging, interactive analysis, and exploit development. Use GDB for remote debugging, multi-threaded applications, scripting, and anything requiring automation.

**Q: Can EDB debug core dumps?**
A: Yes. Open a core dump file directly: `edb --core core.dump`. EDB will display the process state at the time of the crash.

**Q: Does EDB support Python scripting?**
A: Not natively. For Python-based debugging automation, use GDB with GEF or Pwndbg.

## Resources

### Official Documentation
- [EDB GitHub Wiki](https://github.com/eteran/edb-debugger/wiki)
- [EDB Compilation Guides](https://github.com/eteran/edb-debugger/wiki/Compiling-(Ubuntu))

### Tutorials and Guides
- [Linux Exploit Development with EDB](https://www.corelan.be/index.php/2011/07/14/exploit-writing-tutorial-part-1-stack-based-overflows/)
- [EDB Cheat Sheet](https://github.com/eteran/edb-debugger/wiki)

### Books
- *Hacking: The Art of Exploitation* by Jon Erickson
- *The Shellcoder's Handbook* by Chris Anley et al.
- *Practical Binary Analysis* by Dennis Andriesse

### Related Tools
- [GDB](../gdb/gdb.md) — The standard Linux debugger
- [GEF](../gef/gef.md) — GDB Enhanced Features for exploit dev
- [OllyDbg](../ollydbg/ollydbg.md) — Windows predecessor/inspiration
- [Radare2](../radare2/radare2.md) — Command-line debugging alternative
