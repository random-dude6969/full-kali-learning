---
title: "MSF nasm_shell"
tool_name: "msf-nasm_shell"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Beginner"
platform: "Linux"
language: "Ruby, Assembly"
license: "BSD-3-Clause"
repository: "https://github.com/rapid7/metasploit-framework"
kali_package: "metasploit-framework"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "assembly"
  - "disassembly"
  - "shellcode-analysis"
  - "metasploit"
date_created: "2010-01-01"
last_updated: "2024-01-15"
author: "Rapid7"
---

# MSF nasm_shell — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

`nasm_shell` is a Metasploit Framework utility that provides an interactive NASM (Netwide Assembler) shell for translating assembly instructions to opcodes and vice versa. It is an essential tool for shellcode developers, exploit writers, and anyone working with x86/x64 assembly language.

### Purpose

When writing or analyzing shellcode, you frequently need to:
- Know the opcode bytes for a specific assembly instruction
- Understand what a sequence of bytes disassembles to
- Find the exact byte representation of JMP, CALL, or MOV instructions
- Calculate relative offsets for jumps and calls

`nasm_shell` answers all these questions interactively.

### Relationship to ShellNoob

ShellNoob provides similar functionality plus format conversion, but `nasm_shell` is the canonical Metasploit tool for quick opcode lookups. Both tools serve the shellcode development workflow.

---

## Chapter 2: Installation & Setup

### Kali Linux

```bash
# nasm_shell is part of the metasploit-framework package
sudo apt update && sudo apt install metasploit-framework
```

**Explanation:** Installs the Metasploit Framework, which includes nasm_shell as a utility.

### Launch nasm_shell

```bash
msf-nasm_shell
```

Or from within msfconsole:

```bash
msfconsole -q -x "nasm_shell"
```

### Verify Installation

```bash
msf-nasm_shell -h
```

Expected output:

```
Usage: /usr/bin/msf-nasm_shell [options]

Interactive NASM shell for translating assembly to opcodes.
```

---

## Chapter 3: Basic Usage

### Assembly to Opcode Conversion

```bash
msf-nasm_shell
nasm > nop
0x90
nasm > ret
0xc3
nasm > push ebp
0x55
nasm > mov ebp, esp
0x89 0xe5
nasm > int 0x80
0xcd 0x80
```

**Explanation:** Each assembly instruction is translated to its corresponding opcode bytes in real-time.

### Opcode to Assembly Conversion

```bash
nasm > 0x90
nop
nasm > 0x31 0xc0
xor eax, eax
nasm > 0x50 0x68 0x2f 0x73 0x68 0x2f
push 0x2f68732f
nasm > 0x68 0x2f 0x62 0x69 0x6e
push 0x6e69622f
```

**Explanation:** Raw opcode bytes are disassembled back into readable assembly instructions — essential for analyzing existing shellcode.

### Multi-Byte Instructions

```bash
nasm > jmp short 0x10
0xeb 0x10
nasm > call 0x12345678
0xe8 0x73 0x56 0x34 0x12
nasm > mov eax, 0x41424344
0xb8 0x44 0x43 0x42 0x41
```

**Explanation:** Shows how jumps, calls, and MOV instructions with immediate values encode — critical for calculating shellcode offsets.

### Common Shellcode Opcodes Reference

| Instruction | Opcode | Bytes |
|-------------|--------|-------|
| `nop` | `0x90` | 1 |
| `ret` | `0xc3` | 1 |
| `int3` | `0xcc` | 1 |
| `push eax` | `0x50` | 1 |
| `pop eax` | `0x58` | 1 |
| `push ebx` | `0x53` | 1 |
| `pop ebx` | `0x5b` | 1 |
| `xor eax, eax` | `0x31 0xc0` | 2 |
| `xor ebx, ebx` | `0x31 0xdb` | 2 |
| `xor ecx, ecx` | `0x31 0xc9` | 2 |
| `xor edx, edx` | `0x31 0xd2` | 2 |
| `mov eax, imm` | `0xb8` | 5 |
| `mov ebx, imm` | `0xbb` | 5 |
| `mov ecx, imm` | `0xb9` | 5 |
| `mov edx, imm` | `0xba` | 5 |
| `int 0x80` | `0xcd 0x80` | 2 |
| `syscall` | `0x0f 0x05` | 2 |
| `jmp short` | `0xeb` | 2 |
| `jmp near` | `0xe9` | 5 |
| `call` | `0xe8` | 5 |
| `nop` (x64) | `0x90` | 1 |
| `ret` (x64) | `0xc3` | 1 |

---

## Chapter 4: Intermediate Usage

### Calculating JMP Offsets

When writing shellcode, jumps are relative — you need the exact offset:

```bash
# Calculate the jump offset
# If shellcode is at offset 0x00, and target is at offset 0x20:
# Offset = target - current - 2 (for the jmp instruction itself)
nasm > jmp short 0x1e
0xeb 0x1e
```

**Explanation:** JMP offsets are calculated relative to the current instruction. The 2-byte JMP SHORT instruction itself occupies space, so subtract 2 from the target offset.

### Analyzing Existing Shellcode

```bash
# Paste shellcode bytes to see what they do
nasm > fc e8 82 00 00 00 60 89 e5 31 c0 64 8b 50 30 8b 52 0c 8b 52 14 8b 72 28
cld
call 0x84
pusha
mov ebp, eax
xor eax, eax
push eax
mov edx, dword ptr fs:[eax+0x30]
mov edx, dword ptr [edx+0xc]
mov edx, dword ptr [edx+0x14]
mov esi, dword ptr [edx+0x28]
```

**Explanation:** Pasting raw shellcode bytes reveals the instruction sequence, helping understand what the shellcode does.

### Finding Bad Character Alternatives

```bash
# If \x00 is a bad character, find alternatives
nasm > xor eax, eax
0x31 0xc0    # No null bytes - good!

nasm > mov eax, 0
0xb8 0x00 0x00 0x00 0x00    # Contains nulls - bad!

# Alternative: use CDQ to zero EDX, then use other registers
nasm > cdq
0x99    # No null bytes - good!
```

**Explanation:** nasm_shell helps identify which instruction variants avoid bad characters, a critical skill for shellcode development.

### x86 vs x64 Differences

```bash
# 32-bit (default)
nasm > int 0x80
0xcd 0x80

# 64-bit syscall (not directly in nasm_shell, but useful reference)
# syscall = 0x0f 0x05
nasm > 0x0f 0x05
syscall

# 64-bit uses different register sizes
nasm > mov rax, 0
0x48 0xc7 0xc0 0x00 0x00 0x00 0x00
```

**Explanation:** 64-bit instructions often have a REX prefix (0x48) and use different system call conventions — `syscall` instead of `int 0x80`.

---

## Chapter 5: Advanced Usage

### Writing Custom Shellcode

```bash
# Step-by-step shellcode construction
nasm > ; execve("/bin/sh", NULL, NULL)
nasm > xor eax, eax
0x31 0xc0
nasm > push eax
0x50
nasm > push 0x68732f2f
0x68 0x2f 0x2f 0x73 0x68
nasm > push 0x6e69622f
0x68 0x2f 0x62 0x69 0x6e
nasm > mov ebx, esp
0x89 0xe3
nasm > push eax
0x50
nasm > push ebx
0x53
nasm > mov ecx, esp
0x89 0xe1
nasm > cdq
0x99
nasm > mov al, 0xb
0xb0 0x0b
nasm > int 0x80
0xcd 0x80
```

**Explanation:** Constructing shellcode instruction-by-instruction with immediate opcode verification ensures each byte is correct.

### Analyzing Decoder Stubs

```bash
# Shikata Ga Nai decoder stub
nasm > d9 74 24 f4 5f 33 c9 b1 0c
fnstenv [esp-0xc]
pop edi
xor ecx, ecx
mov cl, 0xc
```

**Explanation:** Understanding encoder decoder stubs is crucial for both creating and detecting encoded payloads.

### NOP Sled Construction

```bash
# NOP sled for buffer overflow
nasm > nop
0x90
# Repeat 64 times for a 64-byte sled
# Total: 64 bytes of 0x90
```

**Explanation:** A NOP sled of `0x90` bytes provides a landing zone for exploitation — any address within the sled slides to the shellcode.

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Shellcode Offset Calculation

During buffer overflow exploitation, calculating the exact offset from the return address to the shellcode:

```bash
# The overflow buffer is 2606 bytes
# The return address overwrite is at offset 2606
# Shellcode starts at offset 2610 (after 4 bytes of padding)
# Need to calculate JMP SHORT offset

nasm > ; If we're at offset 2610 and shellcode is at 2630
nasm > jmp short 0x1a
0xeb 0x1a    # Jump 26 bytes forward (2630 - 2610 - 2)
```

**Explanation:** Precise offset calculation prevents shellcode corruption and ensures reliable exploitation.

### Scenario 2: Reverse Engineering Unknown Shellcode

```bash
# Found suspicious bytes in a network capture
nasm > fc e8 82 00 00 00 60 89 e5
cld
call 0x84
pusha
mov ebp, eax

# Continue decoding...
```

**Explanation:** Interactive disassembly helps analysts understand unknown shellcode found in malware samples or network traffic.

### Scenario 3: Custom Encoder Development

Building a custom XOR encoder requires understanding the decoder stub at the opcode level:

```bash
# XOR decoder stub for single-byte XOR
nasm > 31 c9
xor ecx, ecx
nasm > 80 34 0f XX
xor byte ptr [edi+ecx], XX
nasm > 41
inc ecx
nasm > 38 c1
cmp cl, al
nasm > 75 f7
jne -9
```

**Explanation:** Writing custom encoders requires precise opcode knowledge to build working decoder stubs.

---

## Chapter 7: Detection & Defense

### NOP Sled Detection

- Large sequences of `0x90` bytes in network traffic or process memory
- Statistical analysis of byte distribution in suspicious payloads

### Decoder Stub Signatures

- Shikata Ga Nai: `d9 74 24 f4` (fnstenv pattern)
- Custom encoders: XOR + loop patterns (`31 c9 ... 80 34 ... 41 ... 75`)

### Defensive Measures

1. **Monitor** for shellcode patterns in network traffic
2. **Analyze** PE files for embedded NOP sleds
3. **Deploy** memory scanning for decoder stub patterns
4. **Use** YARA rules to detect known encoder signatures
5. **Implement** network IDS signatures for common shellcode byte patterns
6. **Enable** process memory scanning for non-image-backed executable regions

### Shellcode Analysis Workflow

When encountering unknown shellcode during incident response:

1. **Extract** the byte sequence from network capture or memory dump
2. **Paste** into nasm_shell to identify instructions
3. **Map** syscall numbers to their functions
4. **Identify** the payload type: reverse shell, bind shell, command execution
5. **Document** the IOCs: IP addresses, ports, encoded patterns
6. **Build** detection rules based on the identified byte patterns

### Encoder Detection Patterns

| Encoder | Detection Pattern | Description |
|---------|------------------|-------------|
| Shikata Ga Nai | `d9 74 24 f4` | FPU fnstenv prefix |
| alpha_mixed | `eb 03 5e eb ...` | JMP/CALL/POP pattern |
| custom XOR | `31 c9 ... 80 34` | XOR loop with counter |
| countdown | `48 48 48 ... cd 80` | Decrement + syscall |

---

## Chapter 8: Troubleshooting

### "Syntax error" in nasm_shell

**Cause:** Invalid assembly syntax.

```bash
# Use Intel syntax (default)
nasm > mov eax, 0x1    # Correct
nasm > movl $0x1, %eax # Wrong (AT&T syntax)
```

**Explanation:** nasm_shell uses NASM syntax (Intel), not GAS (AT&T). Use Intel syntax for all instructions.

### Unknown Opcodes

**Cause:** The opcode sequence doesn't decode to a valid instruction.

```bash
# Check if it's a multi-byte prefix
nasm > 0x66 0x90
xchg ax, ax    # 0x66 is an operand size prefix
```

**Explanation:** Some bytes are prefixes that modify the following instruction. Understanding x86 prefix bytes prevents misinterpretation.

---

## Resources

- [NASM Documentation](https://www.nasm.us/doc/) — Official NASM manual
- [x86 Opcode Reference](http://ref.x86asm.net/) — Comprehensive opcode table
- [Shell-Storm Shellcode Database](http://shell-storm.org/shellcode/) — Community shellcode
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/) — MSF reference

---

*Last updated: July 2026*
