---
title: "ShellNoob"
tool_name: "shellnoob"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Beginner to Intermediate"
platform: "Linux"
language: "Python"
license: "MIT"
repository: "https://github.com/reyammer/shellnoob"
kali_package: "shellnoob"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "shellcode-writing"
  - "assembly"
  - "format-conversion"
  - "debugging"
date_created: "2013-06-08"
last_updated: "2014-01-21"
author: "Yanick Fratantonio (reyammer)"
---

# ShellNoob — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

ShellNoob is a shellcode writing toolkit that eliminates the tedious, error-prone parts of shellcode development. It converts shellcode between multiple formats, provides interactive assembly-to-opcode conversion, resolves system call numbers, and includes built-in debugging shortcuts.

### Key Features

- **Format conversion**: asm ↔ bin ↔ hex ↔ obj ↔ exe ↔ C ↔ Python ↔ Ruby ↔ bash ↔ safeasm ↔ completec ↔ shellstorm
- **Interactive mode**: Real-time assembly ↔ opcode translation
- **System call reference**: Resolve syscall numbers, constants, and errno values
- **Multi-architecture**: Linux x86, x64, ARM; FreeBSD x86, x64
- **Syntax support**: Both AT&T and Intel assembly syntax
- **Built-in debugging**: Compile-and-debug with strace or gdb in one command
- **Binary patching**: File offset patching, VM address patching, fork nopping
- **Library mode**: Import ShellNoob as a Python module in custom scripts

### Single-Script Deployment

ShellNoob is a single Python script (`shellnoob.py`) with no external dependencies beyond Python, gcc, as, and objdump — all standard on Linux systems.

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
sudo apt update && sudo apt install shellnoob
```

**Explanation:** Installs ShellNoob from the Kali repository. Requires Python 3.

### From Source

```bash
git clone https://github.com/reyammer/shellnoob.git
cd shellnoob
./shellnoob.py --install
```

**Explanation:** Clones the repository and installs the script to `/usr/local/bin/snoob`.

### Verify Installation

```bash
shellnoob -h
# Or if installed via source:
snoob -h
```

Expected output:

```
shellnoob.py [--from-INPUT] (input_file_path | -) [--to-OUTPUT] [output_file_path | - ]
shellnoob.py -c (prepend a breakpoint)
shellnoob.py --64 (64 bits mode, default: 32 bits)
shellnoob.py --intel (intel syntax mode, default: att)
shellnoob.py -q (quite mode)
shellnoob.py -v (or -vv, -vvv)
```

---

## Chapter 3: Basic Usage

### Format Conversion

```bash
# Assembly to raw binary
snoob --from-asm shell.asm --to-bin shell.bin

# Assembly to C array
snoob shell.asm --to-c shell.c

# Assembly to Python
snoob shell.asm --to-python shell.py

# Binary to hex string
snoob --from-bin shell.bin --to-hex shell.hex

# Pipe-based workflow
cat shell.asm | snoob --from-asm - --to-bin - > shell.bin
```

**Explanation:** ShellNoob converts between any supported input and output format, guessing formats from file extensions when possible.

### All Supported Formats

| Format | Description | As Input | As Output |
|--------|-------------|----------|-----------|
| **asm** | Standard assembly (ATT by default) | Yes | Yes |
| **bin** | Raw binary (`\x41\x42`) | Yes | Yes |
| **hex** | Hex-encoded binary (`41424344`) | Yes | Yes |
| **obj** | ELF object file | Yes | Yes |
| **exe** | Executable ELF | No | Yes |
| **c** | C array (`unsigned char buf[]`) | Yes | Yes |
| **completec** | Compilable C with RWX memory setup | No | Yes |
| **python** | Python byte string | No | Yes |
| **bash** | Bash string | No | Yes |
| **ruby** | Ruby byte string | No | Yes |
| **pretty** | Annotated disassembly | No | Yes |
| **safeasm** | 100% assemblable output | No | Yes |
| **shellstorm** | Download from shell-storm DB | Yes | No |

### Interactive Mode

```bash
snoob -i --to-opcode
```

Interactive session:

```
asm_to_opcode selected (type "quit" or ^C to end)
>> xchg %eax, %esp
xchg %eax, %esp ~> 94
>> ret
ret ~> c3
>> nop
nop ~> 90
>> mov %eax, %ebx
mov %eax, %ebx ~> 89c3
>> quit
```

**Explanation:** Interactive mode translates assembly to opcodes in real-time, essential for on-the-fly shellcode debugging.

### Reverse: Opcode to Assembly

```bash
snoob -i --to-asm
```

Interactive session:

```
opcode_to_asm selected (type "quit" or ^C to end)
>> 90
90 ~> nop
>> 89c3
89c3 ~> mov %eax,%ebx
>> c3
c3 ~> ret
>> 31c0
31c0 ~> xor %eax,%eax
>> quit
```

**Explanation:** Reverse conversion reveals what raw byte sequences decode to, crucial for analyzing existing shellcode.

### System Call Numbers

```bash
snoob --get-sysnum read
# i386 ~> 3
# x86_64 ~> 0

snoob --get-sysnum fork
# i386 ~> 2
# x86_64 ~> 57

snoob --get-sysnum execve
# i386 ~> 11
# x86_64 ~> 59
```

**Explanation:** Resolves system call names to their numeric values for both 32-bit and 64-bit architectures — eliminates looking up syscall tables.

### Constants and Error Numbers

```bash
snoob --get-const O_RDONLY
# O_RDONLY ~> 0

snoob --get-const O_CREAT
# O_CREAT ~> 64

snoob --get-errno EACCES
# EACCES ~> Permission denied

snoob --get-errno 13
# 13 ~> Permission denied
```

**Explanation:** Resolves Linux constants and error codes to their values, and vice versa — essential for building syscalls manually.

---

## Chapter 4: Intermediate Usage

### Intel vs AT&T Syntax

```bash
# AT&T syntax (default)
snoob --from-asm shell_att.asm --to-bin shell.bin

# Intel syntax
snoob --intel --from-asm shell_intel.asm --to-bin shell.bin
```

**Explanation:** AT&T syntax uses `%` prefixes and `src, dst` order. Intel syntax uses `dst, src` order and no prefixes. Use whichever matches your shellcode source.

### 32-bit vs 64-bit Mode

```bash
# 32-bit (default)
snoob shell.asm --to-exe shell

# 64-bit
snoob --64 shell.asm --to-exe shell
```

**Explanation:** The `--64` flag enables x86_64 mode, which uses 64-bit registers and different syscall conventions.

### Easy Debugging

```bash
# Compile and debug with gdb
snoob --to-gdb shell.asm
# Launches gdb with breakpoint set at the entry point

# Compile and trace with strace
snoob --to-strace shell.asm
# Compiles and runs the shellcode under strace
```

**Explanation:** These shortcuts eliminate the manual compilation and debugger setup process — ShellNoob handles the entire pipeline.

### Prepend Breakpoint

```bash
snoob -c shell.asm --to-exe shell
```

**Explanation:** The `-c` flag prepends a `int3` breakpoint instruction at the beginning, pausing execution in a debugger for step-by-step analysis.

### Binary Patching Plugins

```bash
# Patch a file at a specific offset
snoob --file-patch target.exe 0x1a2b "\xcc\x90\x90"

# Patch at a virtual memory address
snoob --vm-patch target.exe 0x00401000 "\xcc\x90\x90"

# NOP out fork() calls
snoob --fork-nopper target.exe
```

**Explanation:** Binary patching plugins enable quick modifications to executables without a hex editor — useful for testing and CTF challenges.

### ShellNoob as a Python Library

```python
from shellnoob import ShellNoob

sn = ShellNoob(flag_intel=True)

# Assembly to hex
hex_output = sn.asm_to_hex('nop; mov ebx,eax; xor edx,edx')
print(hex_output)  # 9089c331d2

# Hex to disassembly
instructions = sn.hex_to_inss('9089c331d2')
print(instructions)  # ['nop', 'mov ebx,eax', 'xor edx,edx']

# Resolve syscall
result = sn.do_resolve_syscall('fork')
print(result)  # i386 ~> 2, x86_64 ~> 57
```

**Explanation:** ShellNoob can be imported into Python scripts for programmatic shellcode manipulation and analysis.

---

## Chapter 5: Advanced Usage

### Reading from stdin / Writing to stdout

```bash
# Pipe-based workflows
cat shellcode.asm | snoob --from-asm - --to-hex - | xclip -selection clipboard

# Read shellcode from stdin
snoob --from-hex - --to-c - <<< "9089c331d2c3"
```

**Explanation:** stdin/stdout support enables ShellNoob to integrate into shell pipelines and scripting workflows.

### Safe Assembly Output

```bash
# Generate 100% assemblable output
snoob --from-bin shell.bin --to-safeasm shell_safe.asm
```

**Explanation:** `safeasm` output uses `.byte` notation for bytes that objdump's disassembly might represent as non-assemblable mnemonics.

### Pretty Print with Annotations

```bash
snoob --from-bin shell.bin --to-pretty shell_annotated.asm
```

**Explanation:** Pretty output includes `.byte` and `.ascii` annotations alongside disassembly, providing context for string-containing shellcode.

### Complete C Output

```bash
snoob --from-asm shell.asm --to-completec shell_full.c
```

**Explanation:** `completec` output generates compilable C that sets memory permissions to RWX, supporting self-modifying shellcode.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Writing Shellcode from Scratch

A CTF challenge required hand-crafted shellcode to exploit a buffer overflow:

```bash
# Write the shellcode
cat << 'EOF' > shell.asm
# execve("/bin/sh", NULL, NULL)
push %edx        # 52
push $0x68732f2f  # 68 2f 2f 73 68
push $0x6e69622f  # 68 2f 62 69 6e
mov %esp, %ebx   # 89 e3
push %edx        # 52
push %ebx        # 53
mov %esp, %ecx   # 89 e1
push %edx        # 52
pop %eax         # 58
mov $0xb, %al   # b0 0b
int $0x80        # cd 80
EOF

# Convert to C array
snoob shell.asm --to-c shellcode.c

# Convert to Python for scripting
snoob shell.asm --to-python shellcode.py

# Debug it
snoob -c shell.asm --to-exe shell_debug
gdb -q shell_debug
```

**Explanation:** ShellNoob streamlines the shellcode writing process: write assembly, convert to any format, and debug in one workflow.

### Scenario 2: Analyzing Unknown Shellcode

```bash
# Extract shellcode from a malware sample
# The bytes were: fc e8 82 00 00 00 60 89 e5 31 c0 64 8b 50 30

# Convert to assembly to understand
echo -n "fce8820000006089e531c0648b5030" | snoob --from-hex - --to-asm -

# Or use interactive mode for step-by-step analysis
snoob -i --to-asm
>> fc
>> e8 82 00 00 00
>> ...
```

**Explanation:** Interactive analysis of unknown shellcode byte-by-byte reveals its functionality without requiring a full disassembler.

### Scenario 3: Quick Syscall Reference

During shellcode development, quickly look up the syscall numbers needed:

```bash
snoob --get-sysnum open      # i386 ~> 5
snoob --get-sysnum read      # i386 ~> 3
snoob --get-sysnum write     # i386 ~> 4
snoob --get-sysnum execve    # i386 ~> 11
snoob --get-sysnum exit      # i386 ~> 1

snoob --get-const O_RDWR     # 2
snoob --get-const O_WRONLY   # 1
```

**Explanation:** Quick syscall lookup eliminates the need to search external references, keeping the developer in the flow.

---

## Chapter 7: Detection & Defense

### Shellcode Detection Patterns

- **NOP sleds**: Large sequences of `0x90` in memory or files
- **Common syscall sequences**: `31 c0 50 68 2f 2f 73 68` (execve pattern)
- **Anti-debug techniques**: `int3`, `ptrace` checks in code

### Defensive Measures

1. **Monitor** for assembly patterns associated with shellcode
2. **Deploy** YARA rules for common shellcode signatures
3. **Enable** ASLR and NX/DEP to prevent shellcode execution
4. **Scan** process memory for non-image-backed executable regions

---

## Chapter 8: Troubleshooting

### "gcc: command not found"

**Cause:** GCC is not installed (required for compilation).

```bash
sudo apt install build-essential
```

**Explanation:** ShellNoob uses GCC for compiling shellcode to executables.

### Assembly Syntax Errors

**Cause:** AT&T vs Intel syntax mismatch.

```bash
# Use --intel for Intel syntax input
snoob --intel --from-asm intel_shell.asm --to-bin shell.bin
```

**Explanation:** Default syntax is AT&T. Use `--intel` flag when working with Intel syntax assembly.

### "Permission denied" on Install

**Cause:** Insufficient privileges for system-wide installation.

```bash
sudo ./shellnoob.py --install
```

**Explanation:** System-wide installation to `/usr/local/bin/` requires root privileges.

### Supported Architectures

| Architecture | Platform | Status |
|-------------|----------|--------|
| x86 | Linux | Full support |
| x86_64 | Linux | Full support |
| ARM | Linux | Partial support |
| x86 | FreeBSD | Full support |
| x86_64 | FreeBSD | Full support |

### Common Conversion Workflows

```bash
# Workflow 1: Write → Compile → Debug
snoob shell.asm --to-exe shell
snoob --to-gdb shell.asm

# Workflow 2: Download → Convert → Embed
snoob --from-shellstorm 837 --to-c shellcode.c
# Embed shellcode.c in your custom loader

# Workflow 3: Analyze → Understand → Recreate
snoob --from-bin unknown.bin --to-pretty annotated.asm
snoob -i --to-opcode  # Recreate specific instructions
```

---

## Resources

- [ShellNoob GitHub](https://github.com/reyammer/shellnoob) — Source code
- [Black Hat Arsenal 2013 Slides](https://media.blackhat.com/us-13/Arsenal/us-13-Fratantonio-ShellNoob-Slides.pdf) — Conference presentation
- [Shell-Storm](http://shell-storm.org/shellcode/) — Shellcode database (integrated with `--from-shellstorm`)
- [NASM Documentation](https://www.nasm.us/doc/) — Assembly language reference

---

*Last updated: July 2026*
