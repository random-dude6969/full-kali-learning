# Capstone Cheatsheet

## Quick Reference Card

### Installation

```bash
sudo apt install capstone-tool libcapstone5 libcapstone-dev python3-capstone
```

### Command-Line Usage (cstool)

```bash
# Basic disassembly
cstool x64 90909090                     # Disassemble x86-64 NOP bytes
cstool x32 5589e5                       # Disassemble x86-32 instructions
cstool arm 04e02de5                     # Disassemble ARM instructions
cstool arm64 fd7bbfa9                   # Disassemble ARM64 instructions
cstool mips 03e00008                    # Disassemble MIPS (little-endian)
cstool mipsbe 03e00008                  # Disassemble MIPS (big-endian)
cstool ppc32 4e800020                   # Disassemble PowerPC
cstool sparc 81c3e008                   # Disassemble SPARC
cstool systemz a711a001                 # Disassemble S390x

# With detailed info
cstool -d x64 554889e5                  # Detailed x86-64 disassembly
cstool -d arm 04e02de5                  # Detailed ARM disassembly

# Show version
cstool -v
```

### Architecture Modes

```bash
# x86 modes
cstool x16 <hex>                        # 16-bit mode
cstool x32 <hex>                        # 32-bit mode
cstool x64 <hex>                        # 64-bit mode
cstool x64att <hex>                     # AT&T syntax

# ARM modes
cstool arm <hex>                        # ARM mode
cstool thumb <hex>                      # Thumb mode
cstool cortexm <hex>                    # Cortex-M extensions
cstool armv8 <hex>                      # ARMv8
cstool thumbv8 <hex>                    # Thumb v8

# MIPS modes
cstool mips <hex>                       # MIPS32 little-endian
cstool mipsbe <hex>                     # MIPS32 big-endian
cstool mips64 <hex>                     # MIPS64 little-endian

# BPF modes
cstool bpf <hex>                        # Classic BPF
cstool ebpf <hex>                       # Extended BPF

# Other architectures
cstool wasm <hex>                       # WebAssembly
cstool evm <hex>                        # Ethereum VM
cstool riscv32 <hex>                    # RISC-V 32-bit
cstool riscv64 <hex>                    # RISC-V 64-bit
cstool tc162 <hex>                      # TriCore
```

### Flags Reference

| Flag | Description |
|------|-------------|
| `-d` | Show detailed instruction info (operands, registers, groups) |
| `-s` | Enable SKIPDATA mode (skip non-instruction bytes) |
| `-u` | Show immediates as unsigned values |
| `-v` | Show version and build info |

### Python Bindings

```python
from capstone import *

# Initialize disassembler
md = Cs(CS_ARCH_X86, CS_MODE_64)
md.detail = True

# Disassemble bytes
code = b"\x55\x48\x89\xe5\x48\x83\xec\x10"
for insn in md.disasm(code, 0x1000):
    print(f"0x{insn.address:x}:\t{insn.mnemonic}\t{insn.op_str}")

# Get instruction groups
for insn in md.disasm(code, 0):
    groups = [md.group_name(g) for g in insn.groups]
    print(f"{insn.mnemonic}: groups={groups}")

# SKIPDATA mode
md.skipdata = True
for insn in md.disasm(b"\x55\x00\x00\x48\x89\xe5", 0):
    print(f"{insn.address:x}: {insn.mnemonic} {insn.op_str}")
```

### Python Architecture Constants

```python
# Architectures
CS_ARCH_X86       # x86
CS_ARCH_ARM        # ARM
CS_ARCH_ARM64      # ARM64/AArch64
CS_ARCH_MIPS       # MIPS
CS_ARCH_PPC        # PowerPC
CS_ARCH_SPARC      # SPARC
CS_ARCH_SYSZ       # SystemZ (S390)
CS_ARCH_XCORE      # XCore
CS_ARCH_M68K       # Motorola 68K
CS_ARCH_TMS320C64X # TMS320C64x
CS_ARCH_EVM        # Ethereum VM
CS_ARCH_RISCV      # RISC-V
CS_ARCH_BPF        # BPF

# Modes
CS_MODE_32         # 32-bit (x86)
CS_MODE_64         # 64-bit (x86)
CS_MODE_16         # 16-bit (x86)
CS_MODE_ARM        # ARM mode
CS_MODE_THUMB      # Thumb mode
CS_MODE_MIPS32     # MIPS32
CS_MODE_MIPS64     # MIPS64
CS_MODE_BIG_ENDIAN # Big endian
CS_MODE_LITTLE_ENDIAN # Little endian
```

### Key Python API Methods

```python
md = Cs(arch, mode)

# Configuration
md.detail = True           # Enable detailed instruction info
md.skipdata = True         # Enable SKIPDATA mode
md.mnemonic_is_valid(m)    # Check if mnemonic is valid

# Disassembly
md.disasm(code, address)   # Generator of CsInsn objects

# CsInsn attributes
insn.address               # Instruction address
insn.id                    # Instruction ID
insn.size                  # Instruction size in bytes
insn.mnemonic              # Instruction mnemonic string
insn.op_str                # Instruction operands string
insn.bytes                 # Raw instruction bytes
insn.reg_name(reg_id)      # Convert register ID to name
insn.group_name(group_id)  # Convert group ID to name
insn.regs_read()           # Set of implicit register reads
insn.regs_write()          # Set of implicit register writes
insn.groups()              # List of instruction groups

# Cleanup
md.close()                 # Free resources
```

### Common Patterns

```python
# Find all CALL instructions
for insn in md.disasm(code, 0):
    if insn.mnemonic == "call":
        print(f"Call at 0x{insn.address:x}: {insn.op_str}")

# Find all conditional jumps
for insn in md.disasm(code, 0):
    if insn.mnemonic.startswith("j") and insn.mnemonic != "jmp":
        print(f"Conditional jump at 0x{insn.address:x}")

# Calculate instruction lengths
total = sum(insn.size for insn in md.disasm(code, 0))
```

### C API Quick Reference

```c
#include <capstone/capstone.h>

csh handle;
cs_insn *insn;

// Open handle
cs_open(CS_ARCH_X86, CS_MODE_64, &handle);

// Enable detail mode
cs_option(handle, CS_OPT_DETAIL, CS_OPT_ON);

// Disassemble
size_t count = cs_disasm(handle, code, size, 0x1000, 0, &insn);

// Process results
for (int i = 0; i < count; i++) {
    printf("0x%"PRIx64":\t%s\t%s\n", 
           insn[i].address, insn[i].mnemonic, insn[i].op_str);
}

// Free and close
cs_free(insn, count);
cs_close(&handle);
```

### Supported Architectures List

| Arch | Constant | Modes |
|------|----------|-------|
| x86 | CS_ARCH_X86 | 16, 32, 64 |
| ARM | CS_ARCH_ARM | ARM, Thumb, Cortex-M, ARMv8 |
| ARM64 | CS_ARCH_ARM64 | AArch64 |
| MIPS | CS_ARCH_MIPS | 32, 64 (LE/BE) |
| PPC | CS_ARCH_PPC | 32, 64 |
| SPARC | CS_ARCH_SPARC | SPARC |
| S390 | CS_ARCH_SYSZ | SystemZ |
| XCore | CS_ARCH_XCORE | XCore |
| M68K | CS_ARCH_M68K | M68K |
| TMS320 | CS_ARCH_TMS320C64X | C64x |
| EVM | CS_ARCH_EVM | Ethereum |
| RISCV | CS_ARCH_RISCV | 32, 64 |
| BPF | CS_ARCH_BPF | Classic, Extended |
| WASM | CS_ARCH_WASM | WebAssembly |
| Tricore | CS_ARCH_TRICORE | TC1xx |
