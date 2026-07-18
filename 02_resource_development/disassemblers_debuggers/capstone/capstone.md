---
tool_name: "capstone"
display_name: "Capstone"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install capstone-tool libcapstone5 python3-capstone"
version: "5.0.7"
github_repo: "https://github.com/capstone-engine/capstone"
kali_tools_page: "https://www.kali.org/tools/capstone/"
difficulty_level: "intermediate"
requires_root: false
gui_available: false
tags: ["reverse-engineering", "disassembly", "framework", "library", "multi-architecture"]
related_tools: ["radare2", "ghidra", "gdb", "rizin", "edb-debugger"]
---

# Capstone Disassembly Framework

## Introduction & Overview

Capstone is a lightweight, multi-architecture disassembly framework written in pure C. It is the de facto standard disassembly engine used by virtually every major reverse engineering tool on the planet — including Radare2, Rizin, GDB, Ghidra, and Binary Ninja. Created by Nguyen Anh Quynh and maintained by a dedicated community, Capstone provides a clean, architecture-neutral API for disassembling machine code across more than 20 hardware architectures.

### What Capstone Actually Does

Capstone takes raw bytes and converts them into human-readable assembly instructions. That is its entire job, and it does it faster and more accurately than any competing library. It does not perform decompilation, symbol resolution, or binary analysis — those tasks belong to the tools that embed Capstone internally. What Capstone provides is raw disassembly output, complete with instruction semantics like implicit register reads and writes.

### When to Use Capstone

- Building custom disassembly tools or scripts
- Analyzing shellcode or malware byte sequences
- Embedding disassembly capabilities into your own applications
- Performing bulk binary analysis across multiple architectures
- Writing exploit development tooling that needs instruction-level detail
- Python-based reverse engineering automation

### When NOT to Use Capstone

- When you need a full reverse engineering environment (use Ghidra or Cutter)
- When you need decompilation to pseudocode (use Ghidra's decompiler)
- When you need a debugger (use GDB or edb-debugger)
- When you only need to disassemble a single x86 binary (use objdump)

### Legal and Ethical Considerations

Capstone is a disassembly library — it does not interact with live systems or exploit vulnerabilities. Using Capstone for reverse engineering is legal in most jurisdictions when applied to software you own, have authorization to analyze, or are analyzing for legitimate security research. Unauthorized reverse engineering of proprietary software may violate the DMCA (US), EUCD (EU), or equivalent local legislation. Always verify compliance with applicable laws before analyzing third-party software.

### Key Concepts

- **Architecture-Neutral API**: A single API surface works identically across all supported architectures. Write once, disassemble anywhere.
- **Instruction Decomposition**: Each disassembled instruction includes mnemonic, operands, register reads/writes, and branch information.
- **SKIPDATA Mode**: Capstone can skip over non-code data bytes when disassembling mixed code/data regions.
- **Bindings**: Official Python, Ruby, Java, Go, Rust, C++, and many other language bindings exist.
- **Thread Safety**: The library is thread-safe by design, enabling concurrent disassembly from multiple threads.

### Supported Architectures

| Architecture | Modes |
|---|---|
| x86 | 16-bit, 32-bit, 64-bit (Intel + AT&T syntax) |
| ARM | ARM, Thumb, Cortex-M, ARMv8 |
| AArch64 | Full 64-bit ARM |
| MIPS | MIPS32, MIPS64 (little/big endian) |
| PowerPC | PPC32, PPC64 |
| RISC-V | RV32G, RV64G |
| SPARC | Full SPARC |
| SystemZ | S390x |
| BPF | Classic BPF, eBPF |
| WebAssembly | WASM bytecode |
| Ethereum VM | Solidity bytecode |
| Plus 12 more | M68K, M680X, SH, TriCore, XCore, MOS65XX, etc. |

## Installation & Setup

### System Requirements

- Linux, macOS, Windows, or BSD
- GCC or Clang C compiler (for source builds)
- No runtime dependencies beyond libc

### Installation via APT (Kali Linux)

```bash
sudo apt update && sudo apt install capstone-tool libcapstone5 libcapstone-dev python3-capstone
```

**Explanation:** This installs the cstool command-line utility, the shared library, development headers, and Python 3 bindings.

### Verify Installation

```bash
cstool -v
```

**Explanation:** Displays the Capstone version and build information, confirming a successful installation.

Expected output:

```
Cstool for Capstone Disassembler Engine v5.0.7
```

### Installation from Source

```bash
git clone https://github.com/capstone-engine/capstone
cd capstone
make
sudo make install
```

**Explanation:** Clones the repository, compiles the library, and installs it system-wide. The library installs to `/usr/local/lib/` by default.

### Python Bindings Installation

```bash
pip3 install capstone
```

**Explanation:** Installs the Python 3 bindings via pip, which wrap the C library for scripting.

### Docker

```bash
docker pull capstone/capstone
```

**Explanation:** Pulls the official Docker image with Capstone pre-installed for isolated analysis environments.

## Basic Usage

### First Run: Disassembling x86-64 Shellcode

```bash
cstool x64 90909090
```

**Explanation:** Disassembles four NOP bytes in x86-64 mode, producing one `nop` instruction per byte.

Output:

```
0909090  nop
0909091  nop
0909092  nop
0909093  nop
```

### Disassembling ARM Code

```bash
cstool arm 04e02de5
```

**Explanation:** Disassembles a 4-byte ARM instruction (`push {lr}`) in ARM mode.

### Disassembling with Detail Information

```bash
cstool -d x64 554889e5
```

**Explanation:** The `-d` flag enables detailed output showing operand sizes, register read/write information, and instruction groups.

Output:

```
0554889e5  push   rbp                     ; RE: reg_write(rbp, rsp, rflags)
    op_count: 1
    operands[0]:
        type: REG (rbp)
        size: 8
        access: WRITE
    groups: PUSH, STACK
```

### Disassembling MIPS Big-Endian

```bash
cstool mipsbe 03e00008
```

**Explanation:** Disassembles a MIPS `jr $ra` instruction in big-endian mode.

### Displaying Version Information

```bash
cstool -v
```

**Explanation:** Shows Capstone version, build configuration, and supported architectures.

## Intermediate Usage

### Python Scripting: Bulk Shellcode Analysis

```python
from capstone import *
from capstone.x86 import *

# Initialize disassembler for x86-64 with detail mode
md = Cs(CS_ARCH_X86, CS_MODE_64)
md.detail = True

# Sample shellcode (x86-64 execve("/bin/sh"))
shellcode = b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91"
shellcode += b"\xd0\x8c\x97\xff\x48\xf7\xdb\x53"
shellcode += b"\x48\x89\xe7\x50\x68\x2b\x2d\x31"
shellcode += b"\x68\x49\x89\xe6\x50\x66\x68\x2d"
shellcode += b"\x68\x72\x73\x74\x49\x89\xf9\x48"
shellcode += b"\x31\xd2\x68\x2f\x2f\x73\x68\x68"
shellcode += b"\x2f\x62\x69\x6e\x89\xe3\x50\x57"
shellcode += b"\x51\x53\x89\xe6\xb0\x3b\x0f\x05"

print(f"[*] Disassembling {len(shellcode)} bytes of shellcode:\n")
for insn in md.disasm(shellcode, 0x1000):
    print(f"  0x{insn.address:x}:\t{insn.mnemonic}\t{insn.op_str}")

print(f"\n[*] Total instructions: {len(list(md.disasm(shellcode, 0x1000)))}")
```

**Explanation:** This script initializes Capstone in detail mode, disassembles a common `execve` shellcode payload, and prints each instruction with its address, mnemonic, and operands.

### Detecting Instruction Groups

```python
from capstone import *

md = Cs(CS_ARCH_X86, CS_MODE_64)
md.detail = True

code = b"\x48\x89\x5c\x24\x08\x48\x89\x6c\x24\x10"

for insn in md.disasm(code, 0):
    groups = [md.group_name(g) for g in insn.groups]
    print(f"  {insn.mnemonic:8s} {insn.op_str:30s} groups: {groups}")
```

**Explanation:** Extracts instruction group classifications (JUMP, CALL, RET, etc.) useful for control flow analysis.

### SKIPDATA Mode for Mixed Code/Data

```python
from capstone import *

md = Cs(CS_ARCH_X86, CS_MODE_64)
md.skipdata = True  # Skip data bytes automatically

mixed = b"\x48\x89\xe5\x00\x00\x00\x00\x48\x83\xec\x10"
for insn in md.disasm(mixed, 0):
    print(f"  0x{insn.address:x}: {insn.mnemonic} {insn.op_str}")
```

**Explanation:** SKIPDATA mode makes Capstone automatically skip over non-instruction bytes when encountering data embedded in code sections.

### Cross-Architecture Analysis Script

```python
from capstone import *

archs = {
    "x86-32":  (CS_ARCH_X86, CS_MODE_32),
    "x86-64":  (CS_ARCH_X86, CS_MODE_64),
    "ARM":     (CS_ARCH_ARM, CS_MODE_ARM),
    "ARM64":   (CS_ARCH_ARM64, CS_MODE_ARM),
    "MIPS32":  (CS_ARCH_MIPS, CS_MODE_MIPS32),
}

code = b"\x55\x48\x89\xe5"  # x86-64: push rbp; mov rbp, rsp

for name, (arch, mode) in archs.items():
    md = Cs(arch, mode)
    print(f"\n=== {name} ===")
    for insn in md.disasm(code, 0):
        print(f"  {insn.mnemonic} {insn.op_str}")
    md.close()
```

**Explanation:** Demonstrates how the same bytes produce different disassembly output depending on the target architecture — a critical concept in binary analysis.

## Advanced Usage

### Building a Binary Pattern Scanner

```python
from capstone import *

def scan_nop_sled(binary_path, arch=CS_ARCH_X86, mode=CS_MODE_64):
    with open(binary_path, "rb") as f:
        code = f.read()

    md = Cs(arch, mode)
    nop_sequences = []
    current_start = None
    current_len = 0

    for insn in md.disasm(code, 0):
        if insn.mnemonic == "nop":
            if current_start is None:
                current_start = insn.address
            current_len += 1
        else:
            if current_len >= 4:
                nop_sequences.append((current_start, current_len))
            current_start = None
            current_len = 0

    if current_len >= 4:
        nop_sequences.append((current_start, current_len))

    return nop_sequences

# Usage
for addr, length in scan_nop_sled("/tmp/suspicious_binary"):
    print(f"NOP sled at 0x{addr:x}, length: {length} bytes")
```

**Explanation:** Scans a binary for NOP sled patterns — a common indicator of exploit shellcode or padding in buffer overflow payloads.

### Integration with Radare2

```bash
r2 -q -c 'pd 20' /bin/ls 2>/dev/null | head -20
```

**Explanation:** Radare2 uses Capstone internally. This shows how Capstone's disassembly integrates into larger RE workflows via embedded tools.

### Custom Instruction Filter

```python
from capstone import *
from capstone.x86 import *

md = Cs(CS_ARCH_X86, CS_MODE_64)
md.detail = True

# Find all instructions that read from RSP (stack operations)
with open("/usr/bin/ls", "rb") as f:
    binary = f.read()

count = 0
for insn in md.disasm(binary[:4096], 0x1000):
    if X86_REG_RSP in insn.regs_read():
        print(f"  0x{insn.address:x}: {insn.mnemonic} {insn.op_str}")
        count += 1
        if count > 20:
            break
```

**Explanation:** Filters disassembly output to find all instructions that implicitly read the stack pointer — useful for identifying stack frame setup and teardown patterns.

## Real-World Scenarios

### Scenario 1: Malware Shellcode Analysis

A SOC analyst receives a suspicious document containing an embedded PE file. The exploit payload is XOR-encoded shellcode.

```python
from capstone import *

def decode_and_disassemble(encoded_shellcode, xor_key=0x37):
    decoded = bytes([b ^ xor_key for b in encoded_shellcode])
    md = Cs(CS_ARCH_X86, CS_MODE_64)
    print(f"[*] Decoded {len(decoded)} bytes of shellcode:\n")
    for insn in md.disasm(decoded, 0):
        print(f"  0x{insn.address:04x}: {insn.mnemonic:8s} {insn.op_str}")
```

**Explanation:** XOR-decodes the shellcode and disassembles it, revealing the actual payload hidden behind basic obfuscation. Capstone's speed makes it practical to decode and analyze shellcode in near-real-time during incident response.

### Scenario 2: Firmware Reverse Engineering

An embedded device researcher needs to disassemble firmware from an IoT device running a MIPS-based SoC.

```bash
cstool mips 03e00008    # jr $ra
cstool mips 27bd0020    # addiu $sp, $sp, 0x20
cstool mips afbf001c    # sw $ra, 0x1c($sp)
```

**Explanation:** When analyzing firmware from MIPS-based IoT devices, Capstone's multi-architecture support eliminates the need for separate tools per architecture. The same API handles x86, ARM, MIPS, and 20+ other ISAs.

### Scenario 3: CTF Challenge - Binary Puzzle

A CTF team needs to identify the architecture and disassemble a mystery binary blob extracted from a challenge.

```python
from capstone import *

def identify_arch(code_bytes):
    results = {}
    configs = [
        ("x86-32", CS_ARCH_X86, CS_MODE_32),
        ("x86-64", CS_ARCH_X86, CS_MODE_64),
        ("ARM", CS_ARCH_ARM, CS_MODE_ARM),
        ("ARM64", CS_ARCH_ARM64, CS_MODE_ARM),
    ]

    for name, arch, mode in configs:
        md = Cs(arch, mode)
        valid = 0
        total = 0
        for insn in md.disasm(code_bytes, 0):
            total += 1
            if insn.mnemonic and insn.mnemonic != "(invalid)":
                valid += 1
        if total > 0:
            results[name] = valid / total
        md.close()

    best = max(results, key=results.get)
    print(f"Most likely architecture: {best} ({results[best]:.0%} valid instructions)")
```

**Explanation:** Attempts to disassemble the same bytes across multiple architectures and scores them by the ratio of valid instructions — a practical technique for identifying unknown binaries in CTF challenges.

### Scenario 4: Bug Bounty - Vulnerability Pattern Detection

```python
from capstone import *

md = Cs(CS_ARCH_X86, CS_MODE_64)
md.detail = True

# Search for dangerous patterns: gets(), strcpy(), sprintf()
dangerous_calls = {"gets", "strcpy", "strcat", "sprintf", "scanf"}

with open("target_binary", "rb") as f:
    code = f.read()

for insn in md.disasm(code, 0):
    if insn.mnemonic == "call" and insn.op_str in dangerous_calls:
        print(f"  VULNERABLE: 0x{insn.address:x}: call {insn.op_str}")
```

**Explanation:** Scans a binary for calls to known-dangerous C library functions that lack bounds checking — a quick first pass in vulnerability assessment during bug bounty engagements.

## Detection & Defense

### How Defenders Detect Capstone Usage

Capstone itself is a library, not a standalone tool, so it leaves no direct footprint on a system. However, tools that embed Capstone (like Radare2, Rizin, or custom scripts) may be detected through:

- **Process monitoring**: Unusual processes accessing binary files for extended analysis
- **File access patterns**: Reading binary files in non-sequential patterns typical of disassembly
- **YARA rules**: Custom rules targeting byte patterns associated with Capstone-compiled tools
- **Syscall monitoring**: `ptrace` and `process_vm_readv` syscalls indicate debugging or analysis
- **Network indicators**: Tools built on Capstone may transmit analysis results over the network

### Mitigation Strategies (Blue Team)

- Deploy EDR solutions that flag disassembly framework libraries in unexpected contexts
- Monitor for Python scripts importing Capstone on production systems
- Create application whitelists that restrict RE tool execution
- Monitor `/usr/local/lib/` for the presence of `libcapstone.so` on non-development systems
- Implement file integrity monitoring on critical system binaries

### Blue Team Response

When Capstone-based tools are detected in an environment:
1. Determine the context: is this a developer workstation or a production system?
2. Check if the activity is authorized security testing or unauthorized analysis
3. Review file access logs to identify which binaries were analyzed
4. Correlate with threat intelligence for known APT groups that use custom Capstone tooling

## Troubleshooting

### Common Errors and Solutions

**Error: `cstool: command not found`**

```bash
sudo apt install capstone-tool
```

**Explanation:** The cstool binary is in a separate package from the library. Install the `capstone-tool` package specifically.

**Error: `ImportError: No module named capstone`**

```bash
sudo apt install python3-capstone
# or
pip3 install capstone
```

**Explanation:** Python cannot find the Capstone bindings. Install them via apt or pip.

**Error: Disassembly produces garbage or invalid instructions**

This typically means the wrong architecture or mode was selected. Verify the target architecture:
- x86 binaries: use `CS_MODE_32` or `CS_MODE_64`
- ARM binaries: use `CS_MODE_ARM` or `CS_MODE_THUMB`
- MIPS: specify endianness with `CS_MODE_BIG_ENDIAN` or `CS_MODE_LITTLE_ENDIAN`

**Error: `illegal instruction` or segfault during disassembly**

Capstone may encounter an invalid instruction sequence. Wrap disassembly in a try-except block:

```python
try:
    for insn in md.disasm(code, 0):
        pass
except CsError as e:
    print(f"Disassembly error: {e}")
```

### Performance Optimization

- Use `CS_OPT_DETAIL` only when you need instruction semantics — it adds overhead
- For bulk analysis, batch disassembly into chunks rather than processing entire files at once
- The C API is 10-100x faster than Python bindings for large-scale analysis
- Enable `CS_OPT_SKIPDATA` for mixed code/data regions to avoid disassembly errors

### FAQ

**Q: What is the difference between Capstone and distorm?**
A: Capstone supports 20+ architectures while distorm is x86-only. Capstone also has better community support and more language bindings. Capstone has won.

**Q: Can Capstone decompile code?**
A: No. Capstone is a disassembler only. For decompilation, use Ghidra's built-in decompiler or r2ghidra.

**Q: Does Capstone support the latest Intel AVX-512 instructions?**
A: Yes, Capstone 5.x supports AVX-512 and other recent x86 extensions.

## Resources

### Official Documentation
- [Capstone Engine Website](https://www.capstone-engine.org/)
- [Capstone GitHub Wiki](https://github.com/capstone-engine/capstone/wiki)
- [API Reference](https://www.capstone-engine.org/lang_python.html)

### Tutorials and Guides
- [Capstone Python Binding Tutorial](https://www.capstone-engine.org/lang_python.html)
- [Practical Binary Analysis (book) — uses Capstone extensively](https://practicalbinaryanalysis.com/)
- [Capstone Quick Start Guide](https://github.com/capstone-engine/capstone/blob/next/docs/README)

### Books
- *Practical Binary Analysis* by Dennis Andriesse
- *The IDA Pro Book* by Chris Eagle (references Capstone for custom tooling)
- *Practical Malware Analysis* by Michael Sikorski

### Related Tools
- [Radare2](../radare2/radare2.md) — uses Capstone as its disassembly backend
- [Rizin](../rizin/rizin.md) — radare2 fork, also uses Capstone
- [Ghidra](../ghidra/ghidra.md) — can use Capstone via plugins
- [GDB](../gdb/gdb.md) — uses Capstone for disassembly commands
- [edb-debugger](../edb-debugger/edb-debugger.md) — uses Capstone for disassembly views
