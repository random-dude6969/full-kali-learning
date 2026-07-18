# ShellNoob Cheatsheet

## Quick Reference

### Installation

```bash
sudo apt update && sudo apt install shellnoob
# Or from source:
git clone https://github.com/reyammer/shellnoob.git
cd shellnoob && ./shellnoob.py --install
# Installed to: /usr/local/bin/snoob
```

### Format Conversion

```bash
# Assembly to binary
snoob shell.asm --to-bin shell.bin

# Assembly to C
snoob shell.asm --to-c shell.c

# Assembly to Python
snoob shell.asm --to-python shell.py

# Binary to hex
snoob --from-bin shell.bin --to-hex shell.hex

# Pipe-based
cat shell.asm | snoob --from-asm - --to-bin - > shell.bin
```

### Supported Formats

| As Input | As Output |
|----------|-----------|
| asm, obj, bin, hex, c, shellstorm | asm, obj, exe, bin, hex, c, completec, python, bash, ruby, pretty, safeasm |

### Interactive Mode

```bash
# Assembly → Opcode
snoob -i --to-opcode
>> nop
nop ~> 90
>> ret
ret ~> c3

# Opcode → Assembly
snoob -i --to-asm
>> 90
90 ~> nop
>> 89c3
89c3 ~> mov %eax,%ebx
```

### Syscall Reference

```bash
snoob --get-sysnum read      # i386 ~> 3, x86_64 ~> 0
snoob --get-sysnum fork      # i386 ~> 2, x86_64 ~> 57
snoob --get-sysnum execve    # i386 ~> 11, x86_64 ~> 59
snoob --get-sysnum exit      # i386 ~> 1, x86_64 ~> 60
```

### Constants & Errors

```bash
snoob --get-const O_RDONLY   # 0
snoob --get-const O_CREAT    # 64
snoob --get-errno EACCES     # Permission denied
snoob --get-errno 13         # Permission denied
```

### Syntax Options

```bash
# AT&T syntax (default)
snoob shell_att.asm --to-bin shell.bin

# Intel syntax
snoob --intel shell_intel.asm --to-bin shell.bin
```

### 32/64-bit Mode

```bash
# 32-bit (default)
snoob shell.asm --to-exe shell

# 64-bit
snoob --64 shell.asm --to-exe shell
```

### Debugging Shortcuts

```bash
# Compile and debug with gdb
snoob --to-gdb shell.asm

# Compile and trace with strace
snoob --to-strace shell.asm

# Prepend breakpoint (int3)
snoob -c shell.asm --to-exe shell
```

### Binary Patching

```bash
# Patch at file offset
snoob --file-patch target.exe 0x1a2b "\xcc\x90\x90"

# Patch at VM address
snoob --vm-patch target.exe 0x00401000 "\xcc\x90\x90"

# NOP out fork() calls
snoob --fork-nopper target.exe
```

### Python Library

```python
from shellnoob import ShellNoob
sn = ShellNoob(flag_intel=True)
sn.asm_to_hex('nop; mov ebx,eax')  # '9089c3'
sn.hex_to_inss('9089c3')           # ['nop', 'mov ebx,eax']
sn.do_resolve_syscall('fork')       # i386 ~> 2
```

### Verbose Mode

```bash
snoob -vvv shell.asm --to-bin shell.bin  # Maximum detail
```
