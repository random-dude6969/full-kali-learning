---
tool_name: "gef"
display_name: "GEF (GDB Enhanced Features)"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install gef"
version: "2026.01"
github_repo: "https://github.com/hugsy/gef"
kali_tools_page: "https://www.kali.org/tools/gef/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["reverse-engineering", "debugger", "exploit-development", "gdb", "ctf"]
related_tools: ["gdb", "pwndbg", "peda", "edb-debugger"]
---

# GEF - GDB Enhanced Features

## Introduction & Overview

GEF (pronounced "Jeff") is a single-file Python script that transforms GDB from a cryptic command-line debugger into a modern, information-rich debugging environment for exploit developers and reverse engineers. Created by hugsy, GEF provides architecture-agnostic commands that display context registers, memory maps, breakpoints, and dozens of other data views — all automatically refreshed after every GDB command. It is the most widely used GDB enhancement in the CTF and exploit development community.

### What GEF Actually Does

GEF wraps GDB's Python API to inject a context display (registers, code, stack, memory mappings) that appears after every stopped event. It adds commands like `checksec`, `heap`, `vmmap`, `search`, `dereference`, and many more — all designed to show the information that exploit developers need without typing ten GDB commands per inspection.

### When to Use GEF

- Exploit development on Linux (buffer overflows, format strings, heap exploits)
- CTF binary exploitation challenges
- Reverse engineering ELF binaries interactively
- Kernel debugging with QEMU
- Learning how memory, registers, and the stack work at runtime

### When NOT to Use GEF

- Remote debugging without local GDB access (use gdbserver with GDB directly)
- GUI-based analysis (use edb-debugger or Cutter)
- Automated analysis pipelines (use GDB scripting without GEF overhead)
- Windows debugging (use WinDbg or x64dbg)

### Legal and Ethical Considerations

GEF is a debugging enhancement — it does not exploit vulnerabilities itself. Using GEF to debug software you own or are authorized to test is legal. GEF is the standard tool in CTF competitions and authorized penetration testing. Unauthorized debugging of production systems may violate computer fraud laws.

### Key Concepts

- **Context Display**: After every breakpoint or step, GEF automatically shows registers, disassembly, stack, and memory maps.
- **Architecture Agnostic**: Works identically on x86, x86-64, ARM, ARM64, MIPS, PowerPC, SPARC, and more.
- **Single File**: GEF is one Python file — `gef.py` — with zero external dependencies.
- **Extensible**: Add custom commands by subclassing `GEFCommand` in Python.
- **Heap Analysis**: Inspect glibc heap structures, chunks, bins, and detect use-after-free or double-free bugs.

## Installation & Setup

### System Requirements

- GDB 10.0 or higher, compiled with Python 3.10+ bindings
- Linux (x86, x86-64, ARM, ARM64, MIPS, PowerPC, SPARC)
- No GUI required — pure terminal

### Installation via APT

```bash
sudo apt install gef
```

**Explanation:** Installs GEF from the Kali Linux repositories. This auto-configures GDB to load GEF on startup.

### One-Line Install (Manual)

```bash
bash -c "$(curl -fsSL https://gef.blah.cat/sh)"
```

**Explanation:** Downloads and installs GEF automatically, configuring `~/.gdbinit` to source GEF on every GDB launch.

### Manual Install

```bash
wget -O ~/.gdbinit-gef.py -q https://gef.blah.cat/py
echo "source ~/.gdbinit-gef.py" >> ~/.gdbinit
```

**Explanation:** Downloads the GEF script and adds the source command to your GDB initialization file.

### Verify Installation

```bash
gdb -q
```

**Explanation:** Launches GDB in quiet mode. GEF's colorful context display appearing at the prompt confirms successful installation.

Expected output:

```
     GEF for linux-x86-64, debuggee pid: 12345
     ────────────────────────────────────────────────
        rax   : 0x0
        rbx   : 0x0
        ...
     ────────────────────────────────────────────────
 gef$
```

### Update GEF

```bash
gef update
```

**Explanation:** Updates GEF to the latest version from within GDB. Must be run from the GDB prompt.

## Basic Usage

### Starting a Debugging Session

```bash
gdb -q /bin/ls
```

**Explanation:** Opens `ls` in GDB with GEF loaded. The context display shows registers, code around EIP/RIP, stack contents, and memory mappings.

### Essential GEF Commands

### checksec — Check Binary Protections

```
gef> checksec
```

**Explanation:** Displays the security mitigations enabled in the binary: NX, PIE, RELRO, Stack Canary, and Fortify Source.

Output:

```
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
    FORTIFY:  Fortified
```

### vmmap — View Memory Layout

```
gef> vmmap
```

**Explanation:** Displays the complete memory layout of the debugged process, showing mapped regions with permissions (rwx for exploitable areas).

Output:

```
    Start                End                Offset   Perm Path
    0x0000555555554000 0x0000555555555000 0x000000 r-- /bin/ls
    0x0000555555555000 0x0000555555559000 0x001000 r-x /bin/ls
    0x00007ffff7dc3000 0x00007ffff7de8000 0x000000 r-- /usr/lib/libc.so.6
    0x00007ffff7de8000 0x00007ffff7f38000 0x025000 r-x /usr/lib/libc.so.6
    0x00007ffffffde000 0x00007ffffffff000 0x000000 rw- [stack]
```

### search — Search Memory

```
gef> search "/bin/sh"
```

**Explanation:** Searches all writable memory regions for the string "/bin/sh" — essential for finding or placing the shell string in exploits.

```
gef> search -p 0x41414141
```

**Explanation:** Searches for the 4-byte pattern 0x41414141 (AAAA) in memory — useful for finding overflow data on the stack.

### dereference — Inspect Pointers

```
gef> dereference $rsp 4
```

**Explanation:** Shows 4 pointer-sized values dereferenced from RSP, following the chain of pointers — invaluable for understanding stack contents.

### heap — Heap Analysis

```
gef> heap chunks
```

**Explanation:** Lists all allocated heap chunks with their sizes, flags, and addresses — critical for heap-based exploit development.

```
gef> heap bins
```

**Explanation:** Shows the contents of glibc's free lists (fastbins, tcache, unsorted bin, small/large bins) — identifies freed chunks for use-after-free attacks.

### info functions — List Functions

```
gef> info functions
```

**Explanation:** Lists all functions with debug symbols in the binary, similar to listing available targets for breakpoints.

### Breakpoints

```
gef> break main
gef> break *0x401234
gef> break *(main+42)
```

**Explanation:** Set breakpoints by function name, absolute address, or offset from a symbol.

### Running and Stepping

```
gef> run
gef> continue
gef> stepi
gef> nexti
```

**Explanation:** `run` starts execution, `continue` resumes to the next breakpoint, `stepi` executes one instruction (into calls), `nexti` executes one instruction (over calls).

## Intermediate Usage

### Pattern Creation and De Bruijn

```
gef> pattern create 200
```

**Explanation:** Generates a 200-byte cyclic pattern for finding buffer overflow offsets. Each 4-byte subsequence is unique.

```
gef> pattern search 0x41366241
```

**Explanation:** Finds the offset of the value 0x41366241 within the cyclic pattern, telling you exactly where EIP was overwritten.

### Exploit Development Helper

```bash
# In GDB with GEF:
gef> checksec                    # Check what protections are enabled
gef> vmmap                      # Find executable regions
gef> search "/bin/sh"            # Find shell string in memory
gef> heap chunks                # Inspect heap state
gef> got                        # Show Global Offset Table
gef> plt                        # Show Procedure Linkage Table
```

**Explanation:** These commands form the core exploit development workflow: check protections, find writable memory, locate useful strings, and understand the GOT/PLT layout.

### GOT and PLT Inspection

```
gef> got
```

**Explanation:** Displays the Global Offset Table with resolved addresses, showing which shared library functions have been dynamically resolved.

```
gef> plt
```

**Explanation:** Shows the Procedure Linkage Table entries — the trampolines used for lazy binding of shared library functions.

### Custom GEF Commands

GEF provides over 50 built-in commands. Key ones:

| Command | Purpose |
|---|---|
| `checksec` | Binary security checks |
| `vmmap` | Memory map display |
| `search` | Memory search |
| `heap` | Heap analysis |
| `dereference` | Pointer dereferencing |
| `got` / `plt` | GOT/PLT inspection |
| `pattern` | De Bruijn pattern tools |
| `telescope` | Memory viewer |
| `aslr` | ASLR status and toggle |
| `canary` | Stack canary inspection |

### Scripting with GEF

```python
# gef_custom.py — Add a custom GEF command
import gef

class VulnSearchCommand(gdb.Command):
    """Search for dangerous function calls"""
    def __init__(self):
        super().__init__("find-vulns", gdb.COMMAND_DATA)

    def invoke(self, arg, from_tty):
        dangerous = ["gets", "strcpy", "sprintf", "scanf"]
        for func in dangerous:
            for addr in gef.get_function_addresses(func):
                print(f"[!] Dangerous call: {func} at {addr:#x}")

VulnSearchCommand()
```

**Explanation:** GEF's Python API allows creating custom commands that integrate seamlessly with the GDB prompt and context display.

### Remote Debugging with GEF

```bash
# On target machine:
gdbserver 0.0.0.0:1234 /path/to/binary

# On attacker machine:
gdb -q
gef> target remote 192.168.1.100:1234
```

**Explanation:** GEF works fully over remote debugging connections, maintaining its context display and all enhanced commands.

## Advanced Usage

### Heap Exploitation with GEF

GEF's heap commands reveal the internal state of glibc's memory allocator:

```
gef> heap set-config arena 0x7ffff7fb1b80
gef> heap chunks --arena 0x7ffff7fb1b80
gef> heap bins tcache
gef> heap bins unsorted
```

**Explanation:** These commands inspect tcache, fastbin, and unsorted bin structures — the core data structures exploited in modern heap attacks like tcache poisoning and house-of techniques.

### Kernel Debugging

```bash
# Start QEMU with GDB stub
qemu-system-x86_64 -s -S -kernel bzImage -initrd initramfs.cpio.gz \
    -append "nokaslr console=ttyS0" -nographic

# Connect from GDB
gdb -q
gef> target remote :1234
gef> kernel-offsets
gef> kernel-vmmap
```

**Explanation:** GEF includes kernel-specific commands for debugging Linux kernel modules and analyzing kernel exploits. Disable KASLR with `nokaslr` for easier analysis.

### ASLR Manipulation

```
gef> aslr
ASLR is currently: Enabled
gef> aslr off
gef> aslr on
```

**Explanation:** Toggle ASLR on/off for the debugged process. Disabling ASLR makes addresses deterministic, essential for developing exploits that require fixed addresses.

### Telescope — Advanced Memory Viewer

```
gef> telescope $rsp 20
```

**Explanation:** Displays 20 pointer-sized values starting from RSP, following dereference chains up to 3 levels deep — visualizes the stack more effectively than raw hex dumps.

### GEF Extras (Community Extensions)

```bash
# Install additional community commands
wget -O /tmp/gef-extras.py https://raw.githubusercontent.com/hugsy/gef-extras/main/gef.py
source /tmp/gef-extras.py
```

**Explanation:** GEF-Extras provides additional community-contributed commands like `tcache` analysis, `got` overflow detection, and more specialized exploit helpers.

### Integration with Pwntools

```python
from pwn import *

# Start GDB with GEF in a separate terminal
p = process("/path/to/vulnerable")
pid = p.pid

# Attach GDB with GEF
os.system(f"tmux split-window -h 'gdb -q -ex \"gef-remote {pid}\" {binary}'")
```

**Explanation:** GEF integrates seamlessly with pwntools for exploit development workflows, combining GEF's analysis with pwntools' payload generation.

## Real-World Scenarios

### Scenario 1: Stack Buffer Overflow Exploit

A CTF challenge provides a vulnerable binary with a stack buffer overflow:

```
gef> checksec
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE

gef> pattern create 200
gef> run <<< $(pattern 200)
# Program crashes with control of RIP

gef> pattern search $rip
[+] Found at offset 72

# Build the exploit
gef> search "/bin/sh"
gef> vmmap
# Construct payload: padding(72) + pop_rdi + binsh_addr + system_addr
```

**Explanation:** GEF's checksec, pattern, and vmmap commands provide all the information needed to develop a stack buffer overflow exploit step by step.

### Scenario 2: Heap Exploitation — UAF

A heap-based use-after-free vulnerability:

```
gef> heap chunks
# See chunk at 0x602010 freed but pointer still held

gef> heap bins unsorted
# Confirm the freed chunk is in the unsorted bin

# Exploit: allocate new object at the same address, overwrite function pointer
gef> heap chunks
# Verify the overwrite

gef> continue
# Shell obtained
```

**Explanation:** GEF's heap inspection commands reveal the allocator state, making it possible to understand and exploit heap vulnerabilities.

### Scenario 3: Reverse Engineering a Packed Binary

```
gef> info file
gef> disassemble $pc, +100
gef> telescope $rsp 10
gef> vmmap
# Identify unpacking stub, set breakpoint at OEP
gef> break *0x401580
gef> continue
# At the original entry point, full binary is unpacked
gef> disassemble main
```

**Explanation:** GEF's dynamic analysis complements static analysis in tools like Ghidra. Step through the unpacking stub to reach the original code.

### Scenario 4: Format String Exploitation

```
gef> search -p 0x41414141
# Find where our input lands on the stack
# Use %p format strings to leak stack values

gef> got
# Find GOT entry to overwrite
gef> vmmap
# Find writable memory for the attack

# Craft format string payload to overwrite printf@GOT with system address
```

**Explanation:** GEF's memory inspection and GOT/PLT display provide the addresses needed for format string exploits.

## Detection & Defense

### How Defenders Detect GEF Usage

GEF itself runs inside GDB, so detection focuses on GDB artifacts:

- GDB process visible in `ps` output
- `/proc/<pid>/status` TracerPid field shows the debugger PID
- ASLR may be modified (reduced entropy during debugging)
- Timing anomalies from single-stepping through code
- Anti-debugging checks: `ptrace(PTRACE_TRACEME)` failure indicates a debugger

### Blue Team Response

- Monitor for GDB attachment to sensitive processes
- Implement anti-debugging checks in critical applications
- Alert on ptrace system calls from unexpected sources
- Monitor for GEF installation artifacts in `~/.gdbinit` and `~/.gdbinit-gef.py`

## Troubleshooting

### Common Errors and Solutions

**Error: `Python Exception <class 'ModuleNotFoundError'>`**

```
gef>gef configgef.debug 1
```

**Explanation:** Enable GEF debug mode to identify which Python module is missing, then install it with pip.

**Error: `Could not start tui mode`**

```bash
sudo apt install libreadline-dev
```

**Explanation:** GDB's TUI mode requires readline. Install the development package.

**Error: GEF not loading on GDB start**

```bash
echo "source ~/.gdbinit-gef.py" >> ~/.gdbinit
```

**Explanation:** Ensure the source line exists in `~/.gdbinit` and the file path is correct.

**Error: GEF commands not found**

```
gef> help
```

**Explanation:** Run `help` to verify GEF loaded. If missing, check GDB version (must be 10.0+) and Python version (must be 3.10+).

### Performance Optimization

- Disable GEF's context display for faster batch operations: `gef context-redirect /dev/null`
- Use `gef disassemble` instead of `disassemble` for faster output
- Set `gef.debug` to 0 in production to suppress debug output

### FAQ

**Q: GEF vs Pwndbg vs PEDA — which is best?**
A: GEF has the cleanest codebase and widest architecture support. Pwndbg has better heap analysis. PEDA is legacy — use GEF or Pwndbg. Pick one and stick with it.

**Q: Does GEF work on macOS?**
A: No. GEF is Linux-only. For macOS debugging, use LLDB with lldbinit.

**Q: Can GEF debug Rust binaries?**
A: Yes, but symbol names are mangled. Use `rustfilt` to demangle output.

## Resources

### Official Documentation
- [GEF Documentation](https://hugsy.github.io/gef/)
- [GEF GitHub](https://github.com/hugsy/gef)
- [GEF Commands API](https://hugsy.github.io/gef/api/)

### Tutorials and Guides
- [Exploit Education with GEF](https://github.com/hugsy/gef#readme)
- [Heap Exploitation Guide](https://github.com/hugsy/gef-extras)
- [TryHackMe GDB Room](https://tryhackme.com/room/binaryanalysis0)

### Books
- *Hacking: The Art of Exploitation* by Jon Erickson
- *The Shellcoder's Handbook* by Chris Anley et al.
- *A Guide to Kernel Exploitation* by Errata Security

### Related Tools
- [GDB](../gdb/gdb.md) — The underlying debugger GEF enhances
- [Pwndbg](https://github.com/pwndbg/pwndbg) — Alternative GDB enhancement
- [PEDA](https://github.com/longld/peda) — Legacy GDB enhancement
- [edb-debugger](../edb-debugger/edb-debugger.md) — GUI debugger alternative
