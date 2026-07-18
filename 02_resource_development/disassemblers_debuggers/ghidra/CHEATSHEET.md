# Ghidra Cheatsheet

## Quick Reference Card

### Installation

```bash
sudo apt install ghidra

# From source (latest)
wget https://github.com/NationalSecurityAgency/ghidra/releases/latest/download/ghidra_12.1.1_PUBLIC_*.zip
unzip ghidra_*.zip && cd ghidra_*/
./ghidraRun
```

### Launch

```bash
ghidra                                   # GUI mode
ghidraRun                                # Alternative launcher
/opt/ghidra/support/analyzeHeadless ...  # Headless mode
```

### Headless Analysis

```bash
# Basic headless analysis
/opt/ghidra/support/analyzeHeadless /tmp/project MyProject \
    -import /path/to/binary \
    -postScript MyScript.py

# With script path
/opt/ghidra/support/analyzeHeadless /tmp/project MyProject \
    -import /path/to/binary \
    -scriptPath /home/user/scripts \
    -postScript analyze.py

# Delete project after analysis
/opt/ghidra/support/analyzeHeadless /tmp/project MyProject \
    -import /path/to/binary \
    -postScript script.py \
    -deleteProject

# Batch import
/opt/ghidra/support/analyzeHeadless /tmp/project LargeProject \
    -import /path/to/binaries/*.exe \
    -postScript batch_analyze.py
```

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Ctrl+E` | Go to address |
| `X` | Show cross-references |
| `L` | Edit label/name |
| `Ctrl+L` | Retype variable |
| `;` | Add comment |
| `Ctrl+Shift+F` | Define function |
| `Ctrl+F` | Search |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |
| `Alt+Left` | Navigate back |
| `Alt+Right` | Navigate forward |
| `Ctrl+Shift+G` | Graph function |
| `Ctrl+Shift+C` | Add equate |

### Interface Panels

| Panel | Purpose |
|-------|---------|
| Listing | Disassembly view with annotations |
| Decompiler | C-like pseudocode output |
| Symbol Tree | Functions, labels, classes |
| Data Type Manager | Structures, enums, typedefs |
| Program Tree | Binary sections |
| Console | Script output and logs |

### Decompiler Navigation

| Action | How |
|--------|-----|
| Decompile function | Click function in Symbol Tree |
| Rename variable | Click variable → `L` |
| Retype variable | Click variable → `Ctrl+L` |
| Navigate to xref | Right-click → References |
| Add comment | Click instruction → `;` |

### Essential GhidraPython Commands

```python
# Get current program
program = getCurrentProgram()

# List all functions
for func in program.getFunctionManager().getFunctions(True):
    print(f"{func.getName()}: {func.getEntryPoint()}")

# Get function by name
funcs = getGlobalFunctions("main")

# Decompile function
from ghidra.app.decompiler import DecompInterface
decomp = DecompInterface()
decomp.openProgram(program)
result = decomp.decompileFunction(func, 0, None)
print(result.getDecompiledFunction().getC())

# Search strings
for ref in getReferencesTo(toAddr(0x402000)):
    print(ref)
```

### Script Location

```
# User scripts
~/ghidra_scripts/

# Ghidra built-in scripts
/opt/ghidra/Ghidra/Features/Base/ghidra_scripts/
```

### Export Data

| Format | Method |
|--------|--------|
| Function list | Python script → CSV |
| Pseudocode | Decompiler → Copy |
| Binary diff | Compare Binaries tool |
| Bookmarks | File → Export |

### Analysis Options

When importing a binary, the analysis dialog offers:

| Option | Purpose |
|--------|---------|
| Analysis Speed | Quick vs Comprehensive |
| Analyze All | Enable/disable auto-analysis |
| Processor | Architecture specification |
| Relocations | Handle relocations |
| Symbols | Load/debug symbols |

### Common Script Patterns

```python
# Find dangerous function calls
dangerous = {"gets", "strcpy", "sprintf", "scanf"}
for func in currentProgram.getFunctionManager().getFunctions(True):
    if func.getName() in dangerous:
        print(f"[!] {func.getName()} at {func.getEntryPoint()}")

# Export strings
with open("/tmp/strings.txt", "w") as f:
    for s in currentProgram.getListing().getDefinedData(True):
        if hasattr(s, "getValue") and isinstance(s.getValue(), str):
            f.write(f"{s.getAddress()}: {s.getValue()}\n")

# Create structure
from ghidra.program.model.data import StructureDataType
struct = StructureDataType("config", 0)
struct.add("magic", getDataTypes()[0], 4)
currentProgram.getDataTypeManager().addDataType(struct, None)
```

### Comparison with Alternatives

| Feature | Ghidra | IDA Pro | Cutter |
|---------|--------|---------|--------|
| Price | Free | $$$$ | Free |
| Decompiler | Yes | Hex-Rays | rz-ghidra |
| Collaborative | Yes | Limited | No |
| Scripting | Java/Python | IDAPython | Rizin |
| Analysis depth | Excellent | Excellent | Good |
| Startup time | Slow | Medium | Fast |
| Learning curve | Medium | High | Low |
