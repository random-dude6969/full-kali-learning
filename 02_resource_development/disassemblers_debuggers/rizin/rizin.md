---
tool_name: "rizin"
display_name: "Rizin"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install rizin"
version: "0.8.2"
github_repo: "https://github.com/rizinorg/rizin"
kali_tools_page: "https://www.kali.org/tools/rizin/"
difficulty_level: "intermediate-to-advanced"
requires_root: false
gui_available: false
tags: ["reverse-engineering", "disassembly", "debugger", "command-line", "rizin"]
related_tools: ["radare2", "cutter", "ghidra", "gdb"]
---

# Rizin

## Introduction & Overview

Rizin is a reverse engineering framework born as a community-driven fork of radare2. Created in 2020 after governance disputes in the radare2 project, Rizin focuses on code cleanliness, usability, and sustainable development. It maintains full command compatibility with radare2 while introducing better code organization, improved documentation, and a healthier contributor ecosystem. With Cutter as its official GUI and rz-ghidra providing decompilation, Rizin has become the recommended successor to radare2 for new projects.

### What Rizin Actually Does

Rizin provides the same comprehensive RE capabilities as radare2 — disassembly, debugging, binary analysis, patching, emulation, and scripting — across 50+ CPU architectures and 80+ file formats. The companion tools (rz-bin, rz-asm, rz-hash, rz-diff, rz-find, rz-gg, etc.) handle specific tasks as standalone utilities.

### When to Use Rizin

- Command-line reverse engineering and binary analysis
- Scripted analysis pipelines (rzpipe for Python, Go, Rust, etc.)
- Binary patching and modification
- Firmware analysis and forensics
- Cross-architecture debugging
- Generating shellcode and exploits
- Automated security scanning with custom scripts

### When NOT to Use Rizin

- GUI-based analysis (use Cutter instead)
- Deep decompilation without the rz-ghidra plugin
- When you need IDA Pro's analysis quality for complex binaries
- Learning RE from scratch (GDB with GEF has better learning resources)

### Legal and Ethical Considerations

Rizin is a passive analysis framework — it reads, disassembles, and patches binary files without exploiting vulnerabilities or interacting with remote targets. Using Rizin for authorized security research, CTF competitions, and educational purposes is fully legal. The framework is distributed under LGPLv3 (libraries) and GPLv3 (tools).

### Key Concepts

- **Seek-Based Navigation**: Like radare2, Rizin navigates by seeking to addresses within the binary.
- **Pipe-Friendly Output**: Commands produce JSON output with the `j` suffix for scripting integration.
- **Configuration Engine**: Extensive `e` command for controlling analysis, display, and behavior.
- **ESIL Emulation**: Evaluate String Intermediate Language for CPU emulation without execution.
- **rzpipe**: Language bindings for Python, Go, Rust, Ruby, Haskell, and OCaml.
- **rizin Package Manager (rizipm)**: Community plugins and extensions.
- **RzIL**: A new intermediate language for semantic analysis, replacing ESIL over time.

## Installation & Setup

### System Requirements

- Linux, macOS, Windows, or BSD
- Meson build system (for source builds)
- C compiler (GCC or Clang)
- Python 3 (for scripting)

### Installation via APT

```bash
sudo apt update && sudo apt install rizin
```

**Explanation:** Installs Rizin and all companion tools from the Kali Linux repositories.

### Install from Source (Recommended)

```bash
git clone https://github.com/rizinorg/rizin
cd rizin
meson setup build
meson compile -C build
sudo meson install -C build
```

**Explanation:** Builds the latest Rizin from source using Meson. This provides the newest features and bug fixes.

### Install via pip (rzpipe)

```bash
pip install rzpipe
```

**Explanation:** Installs the Python bindings for programmatic Rizin interaction without needing the full Rizin installation.

### Verify Installation

```bash
rizin -v
```

**Explanation:** Displays the Rizin version and build information, confirming successful installation.

Expected output:

```
rizin 0.8.2 0 @ linux-x86-64
```

### Companion Tools

| Tool | Purpose |
|---|---|
| `rizin` | Main RE framework |
| `rz-bin` | Binary information extractor |
| `rz-asm` | Assembler and disassembler |
| `rz-hash` | Hashing utility |
| `rz-diff` | Binary diffing |
| `rz-find` | Binary pattern search |
| `rz-gg` | Shellcode generator |
| `rz-run` | Program execution wrapper |
| `rz-sign` | FLIRT signature tool |
| `rz-ax` | Base converter |

## Basic Usage

### Opening a Binary

```bash
rizin /usr/bin/ls
```

**Explanation:** Opens the `ls` binary in Rizin's interactive prompt, ready for analysis.

### Automatic Analysis

```
[0x00005a20]> aaa
```

**Explanation:** Performs full automatic analysis — identifies functions, applies autonaming, finds strings, and maps cross-references.

### Essential Commands Reference

| Command | Purpose |
|---|---|
| `aaa` | Full automatic analysis |
| `afl` | List all functions |
| `pdf @main` | Disassemble at main |
| `axt @main` | Cross-references to main |
| `iz` | List strings |
| `ii` | List imports |
| `iS` | List sections |
| `ie` | List entry points |
| `px 64` | Hex dump 64 bytes |
| `s main` | Seek to main |
| `?` | Help (use `cmd?` for command-specific help) |

### Disassembling Functions

```
[0x00005a20]> afl
0x00005a20  42  1024  main
0x00005e30  28  128   sym.check_password

[0x00005a20]> pdf @main
```

**Explanation:** Lists functions with addresses and sizes, then disassembles the target function.

### Listing Strings

```
[0x00005a20]> iz
addr=0x00005f00 section=.rodata string="Hello, World!"
addr=0x00005f10 section=.rodata string="Access Denied"
```

**Explanation:** Extracts and displays all strings from the binary's data sections.

### Hex Dump

```
[0x00005a20]> px 128
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00005a20  5548 89e5 4883 ec20 897d fc89 75f8  UH..H.. .}..u..
```

**Explanation:** Displays 128 bytes in hex format with ASCII representation at the current seek address.

### Running in Script Mode

```bash
rizin -q -c 'aaa; afl; iz' /usr/bin/ls
```

**Explanation:** Executes Rizin commands non-interactively and exits — ideal for scripting and automation.

## Intermediate Usage

### Seek System

```
[0x00005a20]> s sym.check_password
[0x00005e30]> pd 20
[0x00005e30]> s-     # undo seek
[0x00005a20]> s+     # redo seek
[0x00005a20]> s.     # show current seek
```

**Explanation:** Rizin's seek system is the navigation backbone. All display commands are relative to the current position.

### Configuration System

```
[0x00005a20]> e asm.arch
x86
[0x00005a20]> e asm.bits
64
[0x00005a20]> e asm.syntax
intel
[0x00005a20]> e scr.color
1
```

**Explanation:** The `e` command reads and sets configuration. Over 500 options control every aspect of Rizin's behavior.

### Scripting with rzpipe (Python)

```python
import rzpipe

r = rzpipe.open("/usr/bin/ls")
r.cmd("aaa")

# Get functions as JSON
functions = r.cmdj("aflj")
for func in functions:
    print(f"{func['name']}: 0x{func['offset']:x} ({func['size']} bytes)")

# Get strings as JSON
strings = r.cmdj("izj")
for s in strings:
    print(f"String: {s['string']}")

r.quit()
```

**Explanation:** rzpipe provides structured JSON output from any Rizin command, making Python integration trivial for automated analysis.

### Binary Information Extraction

```bash
rz-bin -I /usr/bin/ls
```

**Explanation:** Displays binary format, architecture, entry point, security properties, and checksums.

```bash
rz-bin -s /usr/bin/ls | head -20
```

**Explanation:** Lists symbols with addresses and types for quick binary overview.

### Binary Diffing

```bash
rz-diff -A -B old_binary new_binary
```

**Explanation:** Compares two binaries at the function level, identifying changes between versions — essential for patch analysis.

### Hashing

```bash
rz-hash -a sha256,md5 suspicious_file
```

**Explanation:** Calculates multiple hash values simultaneously for malware identification.

### Pattern Search

```bash
rz-find -x 90909090 /usr/bin/ls
```

**Explanation:** Searches for hex byte patterns within the binary.

### Shellcode Generation

```bash
rz-gg -a x86 -b 32 -f python -i execve
```

**Explanation:** Generates x86-32 shellcode in Python format for exploit development.

### Base Conversion

```bash
rz-ax 10             # decimal to hex
rz-ax 0xa            # hex to decimal
rz-ax -e 0x41        # swap endianness
rz-ax -E "hello"     # base64 encode
rz-ax -D "aGVsbG8=" # base64 decode
```

**Explanation:** rz-ax is a powerful number format converter for working with addresses, hashes, and encoded data.

## Advanced Usage

### ESIL Emulation

```
[0x00005a20]> aei      # initialize ESIL VM
[0x00005a20]> aeim     # initialize ESIL memory
[0x00005a20]> aes      # single step one instruction
[0x00005a20]> aer      # show all registers
[0x00005a20]> aer rax  # show specific register
[0x00005a20]> aec      # continue emulation
```

**Explanation:** ESIL emulates CPU instructions in a virtual environment, enabling analysis of code paths without executing the actual binary.

### RzIL (Intermediate Language)

```
[0x00005a20]> aef @main
```

**Explanation:** Lifts function to RzIL for semantic analysis — a modern replacement for ESIL that provides cleaner semantics for analysis.

### Writing rizin Scripts

```bash
# analysis.rz
aaa
e scr.color=0
afl
iz
pdf @main
```

```bash
rizin -i analysis.rz -q /usr/bin/ls
```

**Explanation:** Rizin scripts automate analysis workflows. The `-i` flag loads and executes the script.

### Binary Patching

```
[0x00005a20]> wa nop
[0x00005a21]> wa nop
[0x00005a22]> wa nop
[0x00005a23]> wa nop
[0x00005a20]> wx 90909090
```

**Explanation:** `wa` writes assembly instructions, `wx` writes raw hex bytes. Use `-w` flag to open in write mode.

### Remote Debugging

```bash
# On target machine
gdbserver 0.0.0.0:1234 /usr/bin/ls

# On analysis machine
rizin -D gdb gdb://192.168.1.100:1234
```

**Explanation:** Rizin connects to GDB server for remote debugging, providing the same interactive analysis environment over the network.

### FLIRT Signatures

```bash
rz-sign -o libc.sig /lib/x86_64-linux-gnu/libc.so.6
```

**Explanation:** Generates FLIRT (Fast Library Identification and Recognition Technology) signatures for identifying known functions in stripped binaries.

### Package Management (rizipm)

```bash
rizipm -s ghidra    # search for ghidra plugin
rizipm -ci rz-ghidra # install ghidra decompiler
rizipm -l            # list installed plugins
```

**Explanation:** rizipm manages community plugins — decompilers, assemblers, analysis modules, and additional tools.

### Advanced Command Patterns

```bash
# Pipe output
afl | grep main

# JSON output and process
aflj | jq '.[].name'

# Address-based commands
pd 20 @ sym.main

# Flag-based operations
f~password     # filter flags
fc password=0x401234  # create flag
```

**Explanation:** Rizin's command system supports Unix-style piping, JSON output, and flag-based address management for complex analysis workflows.

## Real-World Scenarios

### Scenario 1: Quick Malware Triage

```bash
rizin -A -q -c 'aaa; afl; iz; iS' suspicious_binary
```

**Explanation:** One-liner analysis pipeline that provides function list, strings, and sections — a complete overview in seconds.

### Scenario 2: CTF Reverse Engineering

```
$ rizin -A crackme
[0x00005a20]> afl~main
[0x00005a20]> pdf @main | head -50
[0x00005a20]> /r 0xdeadbeef
[0x00005a20]> iz~flag
[0x00005a20]> / string password
```

**Explanation:** Standard CTF workflow: analyze, navigate, search for values and strings, find the flag.

### Scenario 3: Binary Forensics Analysis

```bash
rz-bin -H suspicious_file
rz-hash -a md5,sha1,sha256 suspicious_file
rz-bin -z suspicious_file | grep -i "http\|ftp\|tcp"
rz-bin -i suspicious_file | grep -i "socket\|connect\|send"
```

**Explanation:** Multi-tool forensic analysis building a complete behavioral profile from binary metadata, hashes, strings, and imports.

### Scenario 4: Firmware Reverse Engineering

```bash
# Identify architecture
file firmware.bin

# Open with correct architecture
rizin -a arm -b 32 firmware.bin

# Analyze
aaa
afl
iz~http
```

**Explanation:** Rizin handles firmware analysis with explicit architecture specification — critical for embedded devices where auto-detection fails.

### Scenario 5: Vulnerability Scanning Script

```python
import rzpipe

dangerous = ["gets", "strcpy", "sprintf", "scanf", "strcat"]

r = rzpipe.open("/usr/bin/target")
r.cmd("aaa")

imports = r.cmdj("iij")
for imp in imports:
    if imp.get("name", "") in dangerous:
        print(f"[!] Vulnerable import: {imp['name']} at 0x{imp['plt']:x}")

r.quit()
```

**Explanation:** Automated vulnerability scanning using rzpipe to identify dangerous function imports — a foundation for custom security analysis tools.

## Detection & Defense

### How Defenders Detect Rizin

- Process name: `rizin`, `rz-asm`, `rz-bin` in process listings
- Shared libraries: `librz_*` libraries loaded into memory
- File access patterns: non-sequential binary reads
- Package management: `dpkg -l | grep rizin`
- User directories: `~/.rizin/` configuration directory

### Blue Team Response

- Monitor for Rizin process execution on production systems
- Alert on rzpipe script execution
- Check for Rizin installation on compromised systems
- Review file access logs for binary analysis patterns

## Troubleshooting

### Common Errors and Solutions

**Error: `rizin: command not found`**

```bash
sudo apt install rizin
```

**Explanation:** Rizin may not be installed. Use apt for quick installation.

**Error: `Cannot find function at address`**

```
[0x00000000]> aaa
```

**Explanation:** Run full analysis first. Many commands require analysis to be performed.

**Error: Slow analysis on large binaries**

```bash
rizin -e analysis.timeout=600 large_binary
```

**Explanation:** Large binaries require extended analysis time. Set appropriate timeouts.

**Error: `No symbols found`**

The binary may be stripped. Use `aaa` for function detection without symbols:

```
[0x00000000]> aaa
[0x00000000]> afl
```

**Explanation:** Rizin's analysis engine identifies functions even in stripped binaries using prologue/epilogue patterns and call graph analysis.

**Error: Wrong architecture detected**

```bash
rizin -a arm -b 32 binary.bin
```

**Explanation:** Manually specify the architecture when auto-detection fails, especially for firmware and cross-architecture binaries.

### Performance Optimization

- Use `-NN` flag to disable plugins for faster startup
- Target analysis with `afb` instead of full `aaa`
- Use JSON output (`j` suffix) for scripted processing
- Pipe through `head` or `grep` for quick filtering

### FAQ

**Q: Rizin vs Radare2 — which is better?**
A: Rizin has cleaner code, better documentation, and more sustainable governance. Radare2 has more plugins and community resources. For new projects, choose Rizin. For legacy scripts, use r2.

**Q: Does Rizin support ARM64 binaries?**
A: Yes. Rizin supports 50+ architectures including x86, x86-64, ARM, ARM64, MIPS, PowerPC, RISC-V, SPARC, and more.

**Q: How do I get decompilation in Rizin?**
A: Install the rz-ghidra plugin: `rizipm -ci rz-ghidra`, then use `pdg` to decompile functions.

## Resources

### Official Documentation
- [Rizin Website](https://rizin.re/)
- [Rizin Book](https://book.rizin.re/)
- [GitHub](https://github.com/rizinorg/rizin)
- [Building Guide](https://github.com/rizinorg/rizin/blob/dev/BUILDING.md)

### Tutorials and Guides
- [Rizin Getting Started](https://rizin.re/install/)
- [Command Reference](https://book.rizin.re/rizin/)
- [rzpipe Documentation](https://github.com/rizinorg/rz-pipe)

### Books
- *Rizin Book* (official, free at book.rizin.re)
- *Practical Binary Analysis* by Dennis Andriesse
- *Reverse Engineering for Beginners* by Dennis Yurichev

### Videos
- [Rizin YouTube Channel](https://www.youtube.com/@rizinorg)
- [RizinCon Conference Talks](https://www.youtube.com/@rizinorg)

### Related Tools
- [Radare2](../radare2/radare2.md) — Rizin's predecessor
- [Cutter](../cutter/cutter.md) — Official Rizin GUI
- [Ghidra](../ghidra/ghidra.md) — NSA RE suite with decompiler
- [GDB](../gdb/gdb.md) — Standard Linux debugger
