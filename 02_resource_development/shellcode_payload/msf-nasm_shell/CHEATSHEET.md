# MSF nasm_shell Cheatsheet

## Quick Reference

### Launch

```bash
msf-nasm_shell
# Or from msfconsole:
msfconsole -q -x "nasm_shell"
```

### Assembly → Opcode

```bash
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

nasm > xor eax, eax
0x31 0xc0

nasm > jmp short 0x10
0xeb 0x10
```

### Opcode → Assembly

```bash
nasm > 0x90
nop

nasm > 0x31 0xc0
xor eax, eax

nasm > 0x50 0x68 0x2f 0x73 0x68 0x2f
push 0x2f68732f

nasm > fc e8 82 00 00 00 60 89 e5
cld
call 0x84
pusha
mov ebp, eax
```

### Common Opcodes

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
| `int 0x80` | `0xcd 0x80` | 2 |
| `syscall` | `0x0f 0x05` | 2 |
| `jmp short` | `0xeb` | 2 |
| `jmp near` | `0xe9` | 5 |
| `call` | `0xe8` | 5 |
| `mov eax, imm` | `0xb8` | 5 |

### JMP Offset Calculation

```bash
# JMP SHORT = 2 bytes total
# Offset = target - current - 2

# Example: from offset 0x00 to offset 0x20
nasm > jmp short 0x1e
0xeb 0x1e
```

### NOP Sled

```bash
nasm > nop
0x90
# Repeat 64 times = 64-byte sled (64 × 0x90)
```

### x86 vs x64

```bash
# x86: int 0x80
nasm > int 0x80
0xcd 0x80

# x64: syscall
nasm > 0x0f 0x05
syscall

# x64: REX prefix (0x48) for 64-bit operands
nasm > 0x48 0x31 0xc0
xor rax, rax
```

### Bad Character Analysis

```bash
# Find alternatives that avoid null bytes
nasm > xor eax, eax
0x31 0xc0    # No nulls ✓

nasm > mov eax, 0
0xb8 0x00 0x00 0x00 0x00  # Has nulls ✗
```
