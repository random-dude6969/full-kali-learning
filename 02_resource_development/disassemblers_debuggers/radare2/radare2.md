---
tool_name: "radare2"
display_name: "Radare2"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install radare2"
version: "6.0.4"
github_repo: "https://github.com/radareorg/radare2"
kali_tools_page: "https://www.kali.org/tools/radare2/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["reverse-engineering", "disassembly", "debugger", "command-line", "forensics"]
related_tools: ["rizin", "cutter", "ghidra", "gdb"]
---

# Radare2

## Introduction & Overview

Radare2 (r2) is a complete reverse engineering framework and command-line toolset. Born as a simple hex editor focused on forensics, r2 evolved into a full-featured RE platform covering disassembly, debugging, binary analysis, patching, scripting, and more. It supports over 50 CPU architectures, 80+ file formats, and can mount filesystems, analyze firmware, and debug remote targets. The project started in 2006 by pancake (Sergi Alvarez) and now has over 24,000 GitHub stars.

### What Radare2 Actually Does

Radare2 is a Swiss Army knife for binary analysis. It opens any file format, disassembles code across 50+ architectures, provides a built-in debugger, patches binaries in-place, compares binary versions, generates shellcode, and wraps everything in a scriptable command system. The companion tools (rabin2, rasm2, rahash2, radiff2, etc.) provide standalone functionality for specific tasks.

### When to Use Radare2

- Quick command-line binary analysis without launching a GUI
- Scripted/automated RE pipelines (r2pipe integration)
- Binary patching and modification
- Firmware analysis and forensics
- Comparing binary versions (patch diffing)
- Generating and testing shellcode (ragg2)
- Hashing and checksumming binary sections

### When NOT to Use Radare2

- GUI-based analysis (use Cutter or Ghidra)
- Deep decompilation (use Ghidra's decompiler)
- Learning RE as a beginner (GDB with GEF is more approachable)
- Very large binaries where analysis time matters (Ghidra handles this better)

### Legal and Ethical Considerations

Radare2 is a passive analysis tool. It reads, analyzes, and patches binary files without exploiting vulnerabilities or interacting with remote systems. Using r2 for authorized security research, CTF challenges, and educational purposes is legal. The framework is distributed under LGPLv3 with permissive plugin licensing.

### Key Concepts

- **Seek-Based Navigation**: r2 uses "seek" commands to navigate the binary, similar to `cd` in a filesystem.
- **Command Verbosity**: Most r2 commands accept prefixes like `p` (print), `a` (analyze), `w` (write), `s` (seek).
- **Config System**: Extensive configuration via `e` commands controls every aspect of behavior.
- **ESIL**: Evaluable String Intermediate Language for emulation without running the binary.
- **r2pipe**: Script any language (Python, Go, Rust, Node.js) to automate r2.
- **Plugins**: Extend r2 via r2pm package manager with hundreds of community plugins.

## Installation & Setup

### System Requirements

- Linux, macOS, Windows, or BSD
- C compiler (GCC or Clang)
- Make or Meson build system
- Terminal (command-line only, no GUI required)

### Installation via APT

```bash
sudo apt update && sudo apt install radare2
```

**Explanation:** Installs radare2 and all companion tools (rabin2, rasm2, rahash2, radiff2, rafind2, ragg2, etc.) from the Kali repositories.

### Install from Git (Recommended for Latest Version)

```bash
git clone https://github.com/radareorg/radare2
cd radare2
sys/install.sh
```

**Explanation:** The recommended installation method. Compiles the latest version from source with all features enabled and installs system-wide.

### Install via pip

```bash
pip install r2env
r2env init
r2env install git+https://github.com/radareorg/radare2
```

**Explanation:** Uses r2env to manage r2 installations without root access. Useful for maintaining multiple versions.

### Verify Installation

```bash
r2 -v
```

**Explanation:** Displays the radare2 version and library versions, confirming successful installation.

Expected output:

```
radare2 6.0.4 0 @ linux-x86-64
```

### Companion Tools

| Tool | Purpose |
|---|---|
| `r2` | Main RE framework |
| `rabin2` | Binary information extractor |
| `rasm2` | Assembler/disassembler |
| `rahash2` | Hashing utility |
| `radiff2` | Binary diffing |
| `rafind2` | Binary pattern search |
| `ragg2` | Shellcode generator |
| `rarun2` | Program execution wrapper |
| `r2pm` | Package manager |

## Basic Usage

### Opening a Binary

```bash
r2 /bin/ls
```

**Explanation:** Opens `ls` in radare2's interactive prompt. The binary is loaded in read-only mode by default.

### Automatic Analysis

```
[0x00401230]> aaa
```

**Explanation:** Runs "analyze all autoname all" — identifies functions, symbols, strings, cross-references, and applies autonaming. This is the essential first step.

### Essential Commands

| Command | Purpose |
|---|---|
| `aaa` | Analyze all (functions, symbols, strings) |
| `afl` | List all functions |
| `pdf @main` | Disassemble function at main |
| `axt sym.main` | Cross-references to main |
| `iz` | List strings |
| `ii` | List imports |
| `ie` | List entry points |
| `px 64` | Hex dump 64 bytes |
| `s sym.main` | Seek to main function |
| `q` | Quit |

### Disassembling a Function

```
[0x00401230]> afl
0x00401230  42  1024  main
0x00401630  28  128   check_password

[0x00401230]> pdf @main
```

**Explanation:** Lists all functions with their addresses and sizes, then disassembles the main function.

### Inspecting Strings

```
[0x00401230]> iz
vaddr=0x00402000 paddr=0x00002000 section=.data string="Access Granted"
vaddr=0x00402010 paddr=0x00002010 section=.data string="Access Denied"
```

**Explanation:** Lists all strings in the binary with their virtual and physical addresses.

### Hex Dump

```
[0x00401230]> px 128
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00401230  5548 89e5 4883 ec20 897d fc89 75f8  UH..H.. .}..u..
0x0040123e  4889 75f0 488b 45f8 0fb6 0084 c074  H.u.H.E ......t
```

**Explanation:** Displays a hex dump with ASCII representation — the fundamental view for binary inspection.

### Running in Script Mode

```bash
r2 -q -c 'aaa; afl; iz' /bin/ls
```

**Explanation:** Runs r2 in quiet mode, executes three commands, and exits. Perfect for scripting and automation.

## Intermediate Usage

### The Seek System

Radare2 navigates by "seeking" to addresses:

```
[0x00401230]> s 0x401630
[0x00401630]>
[0x00401630]> s-  # undo seek
[0x00401230]> s+  # redo seek
```

**Explanation:** Think of seeking as `cd` in a filesystem. All display commands show data relative to the current seek position.

### Configuration System

```
[0x00401230]> e asm.arch
x86
[0x00401230]> e asm.bits
64
[0x00401230]> e scr.color
1
[0x00401230]> e asm.syntax=intel
```

**Explanation:** The `e` command reads and sets configuration variables. Hundreds of options control architecture, display, analysis, and behavior.

### Scripting with r2pipe (Python)

```python
import r2pipe

r2 = r2pipe.open("/bin/ls")
r2.cmd("aaa")

# Get function list as JSON
functions = r2.cmdj("aflj")
for func in functions:
    print(f"{func['name']}: 0x{func['offset']:x} ({func['size']} bytes)")

# Get strings as JSON
strings = r2.cmdj("izj")
for s in strings:
    print(f"String at 0x{s['vaddr']:x}: {s['string']}")

r2.quit()
```

**Explanation:** r2pipe provides JSON-structured output from any r2 command, making it trivial to integrate r2 analysis into Python scripts.

### Binary Information Extraction

```bash
rabin2 -I /bin/ls
```

**Explanation:** Displays binary information: format, architecture, entry point, checksums, and security properties.

```bash
rabin2 -s /bin/ls | head -20
```

**Explanation:** Lists symbols in the binary with addresses and types.

### Binary Diffing

```bash
radiff2 -C old_version.exe new_version.exe
```

**Explanation:** Compares two binaries at the code level, showing matching and differing functions — essential for patch analysis and vulnerability research.

### Hashing

```bash
rahash2 -a md5 -a sha256 suspicious_binary
```

**Explanation:** Calculates multiple hash values for a binary — MD5, SHA256, and others for malware identification and IOC creation.

### Pattern Search

```bash
rafind2 -x 90909090 /bin/ls
```

**Explanation:** Searches for hex patterns (NOP sleds in this case) within the binary.

### Shellcode Generation

```bash
ragg2 -a x86 -b 32 -f python -i execve
```

**Explanation:** Generates x86-32 shellcode for the execve syscall in Python format — useful for exploit development.

### Program Execution Wrapper

```bash
rarun2 arg1=hello arg2=world chroot=/tmp /path/to/binary
```

**Explanation:** rarun2 runs a program with controlled environment variables, arguments, and system settings — useful for debugging with specific configurations.

## Advanced Usage

### ESIL (Emulation)

ESIL allows executing instructions in r2's emulator without running the actual binary:

```
[0x00401230]> aei       # init ESIL VM
[0x00401230]> aeim      # init ESIL memory
[0x00401230]> aes       # single step
[0x00401230]> aer rax   # show register
```

**Explanation:** ESIL emulates CPU instructions, enabling analysis of code paths that are difficult to reach through normal execution.

### Writing r2 Scripts

```bash
# analysis.r2 — Automated analysis script
aaa
e scr.color=0
afl~FUNC~main
iz~string
axt@main
```

```bash
r2 -i analysis.r2 -q /path/to/binary
```

**Explanation:** r2 scripts automate repetitive analysis tasks. The `-i` flag loads and executes a script file at startup.

### Binary Patching

```
[0x00401230]> wa nop
[0x00401231]> wa nop
[0x00401232]> wa nop
[0x00401233]> wa nop
[0x00401230]> ww hello\x00 @ 0x402000
```

**Explanation:** The `wa` command writes assembly instructions, `ww` writes raw bytes. Patching modifies the binary in-place (open with `-w` flag).

### Remote Debugging

```bash
# Target machine
r2 -D gdb -c 'doo' /path/to/binary  # or use gdbserver
gdbserver 0.0.0.0:1234 /path/to/binary

# Analysis machine
r2 -D gdb gdb://192.168.1.100:1234
```

**Explanation:** r2 can debug remote targets using GDB's remote protocol, connecting to gdbserver over TCP.

### Plugin Management (r2pm)

```bash
r2pm -s ghidra    # search for ghidra-related plugins
r2pm -ci r2ghidra  # install ghidra decompiler plugin
r2pm -l            # list installed plugins
```

**Explanation:** r2pm manages community plugins for extending r2's capabilities — decompilers, assemblers, analysis modules, and more.

### Advanced Command Patterns

```bash
# Filter output
afl~sym.main

# JSON output
aflj | jq '.[].name'

# Combine commands
aaa; afl~main; pdf @$$

# Use temporary files
wtf /tmp/dump.bin 0x1000  # dump bytes to file
```

**Explanation:** r2's command system supports piping (`~` for grep, `@` for addressing, `;` for chaining), JSON output, and integration with standard Unix tools.

## Real-World Scenarios

### Scenario 1: Quick Malware Triage

```bash
r2 -A -q -c 'aaa; afl; iz; iS' suspicious_binary
```

**Explanation:** One-liner automatic analysis that lists functions, strings, and sections — providing a quick overview of the binary's structure and behavior.

### Scenario 2: CTF Reverse Engineering

```
$ r2 -A crackme
[0x00401230]> afl~main
0x00401230  142  5232  main
[0x00401230]> pdf @main | head -30
[0x00401230]> /r 0xdeadbeef  # search for magic values
[0x00401230]> iz~flag        # search for flag strings
```

**Explanation:** Navigate to the main function, examine the disassembly, search for magic values, and look for flag-related strings — the standard CTF workflow.

### Scenario 3: Patch Diffing for Vulnerability Discovery

```bash
radiff2 -s -A -B old_lib.so new_lib.so
```

**Explanation:** Compares two library versions at the function level, identifying which functions changed and where — the first step in discovering what a security patch fixed.

### Scenario 4: Binary Forensics

```bash
rabin2 -H suspicious_file
rahash2 -a md5,sha1,sha256 suspicious_file
rabin2 -z suspicious_file | grep -i "http\|ftp\|tcp"
rabin2 -i suspicious_file | grep -i "socket\|connect\|send"
```

**Explanation:** Multi-tool forensic analysis combining binary information, hashing, string extraction, and import analysis to build a complete profile of a suspicious file.

## Detection & Defense

### How Defenders Detect Radare2

- Process name: `r2`, `rasm2`, `rabin2` visible in process listings
- Shared libraries: `libr_*` libraries loaded into memory
- File access patterns: non-sequential binary reads
- Network indicators: r2 agent server (`r2agent`) listening on ports
- Package management: `dpkg -l | grep radare2`

### Blue Team Response

- Monitor for radare2 process execution on non-development systems
- Alert on r2agent HTTP server startup (port 8080 by default)
- Check for radare2 installation on compromised systems
- Review file access logs for binary analysis patterns

## Troubleshooting

### Common Errors and Solutions

**Error: `r2: command not found`**

```bash
sudo apt install radare2
# or install from git for latest version
git clone https://github.com/radareorg/radare2
cd radare2 && sys/install.sh
```

**Explanation:** The Kali package may be outdated. Installing from git provides the latest version with bug fixes.

**Error: `Cannot find function at address`**

Run full analysis first:

```
[0x00000000]> aaa
```

**Explanation:** Many r2 commands require analysis to be performed first. Always start with `aaa` or `aaaa` for thorough analysis.

**Error: Slow analysis on large binaries**

```bash
r2 -e analysis.timeout=600 -A large_binary
```

**Explanation:** Large binaries take time to analyze. Set a longer timeout and be patient. For very large binaries, use targeted analysis on specific functions.

**Error: Plugin conflicts**

```bash
r2pm -U       # Update database
r2pm -r r2    # Reload plugins
r2pm -l       # List installed plugins
```

**Explanation:** Plugin conflicts can cause crashes. Update and reload plugins, or temporarily disable them with `-NN` flag.

### Performance Optimization

- Use `-NN` flag to disable all plugins for maximum speed
- Target analysis with `afb` (analyze function blocks) instead of `aaa`
- Use JSON output (`j` suffix) for scripted processing
- Pipe output through `head` or `grep` for quick filtering

### FAQ

**Q: Radare2 vs Rizin — which should I use?**
A: Rizin is the fork with better code quality, more features, and active development. Radare2 has more plugins and community resources. Start with Rizin for new projects; use r2 for legacy scripts.

**Q: How do I learn r2's command syntax?**
A: Start with the official book (book.rada.re), then practice with CTF challenges. The `?` suffix shows help for any command (e.g., `af?` for analysis function help).

**Q: Can r2 decompile code?**
A: Install the r2ghidra plugin: `r2pm -ci r2ghidra`, then use `pdg` to decompile. The decompiler is based on Ghidra's p-code engine.

## Resources

### Official Documentation
- [Radare2 Website](https://www.radare.org/)
- [Radare2 Book](https://book.rada.re/)
- [GitHub](https://github.com/radareorg/radare2)
- [USAGE.md](https://github.com/radareorg/radare2/blob/master/USAGE.md)

### Tutorials and Guides
- [Radare2 Cheat Sheet](https://github.com/radareorg/radare2/blob/master/doc/intro.md)
- [r2pipe Tutorial](https://book.rada.re/r2/r2pipe.html)
- [Radare2 for Beginners](https://www.youtube.com/watch?v=OllyDbg_tutorial)

### Books
- *Radare2 Book* (official, free at book.rada.re)
- *Practical Binary Analysis* by Dennis Andriesse
- *The IDA Pro Book* by Chris Eagle

### Videos
- [r2con Conference Talks](https://www.youtube.com/c/r2con)
- [LiveOverflow Radare2 Series](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)

### Related Tools
- [Rizin](../rizin/rizin.md) — Rizin fork with cleaner codebase
- [Cutter](../cutter/cutter.md) — GUI for Radare2/Rizin
- [Ghidra](../ghidra/ghidra.md) — NSA RE suite with decompiler
- [GDB](../gdb/gdb.md) — Standard Linux debugger
