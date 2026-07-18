# Cutter Cheatsheet

## Quick Reference Card

### Installation

```bash
# Kali Linux
sudo apt install cutter-re

# AppImage (latest)
wget https://github.com/rizinorg/cutter/releases/latest/download/Cutter-x86_64.AppImage
chmod +x Cutter-x86_64.AppImage

# Build from source
git clone --recurse-submodules https://github.com/rizinorg/cutter.git
cd cutter && mkdir build && cd build
cmake .. && make -j$(nproc)
```

### Launch

```bash
cutter /path/to/binary                    # Open binary directly
cutter                                    # Launch and select file
cutter -w binary.exe                      # Open in write mode
cutter -e analysis.timeout=600 binary     # Extended analysis timeout
```

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `F5` | Graph View |
| `F2` | Toggle Breakpoint |
| `Ctrl+Shift+F` | Find string |
| `Space` | Disassemble at cursor |
| `G` | Go to address |
| `N` | Rename function/label |
| `X` | Cross-references |
| `;` | Add comment |
| `Alt+Left` | Navigate back |
| `Alt+Right` | Navigate forward |

### Interface Panels

| Panel | Shortcut | Purpose |
|-------|----------|---------|
| Disassembly | `F5` toggle | View disassembled instructions |
| Graph | `F5` | Control flow graph |
| Hexdump | — | Raw binary view |
| Strings | — | Extracted strings table |
| Functions | — | All identified functions |
| Imports | — | External dependencies |
| Exports | — | Exported symbols |
| Decompiler | — | Pseudocode output (rz-ghidra) |
| Console | — | Integrated Rizin terminal |

### Essential Rizin Commands (via Console)

```
# In Cutter's console panel:
afl                     # List all functions
pdf @main               # Disassemble at main
axt @main               # Cross-references to main
iz                      # List strings
ii                      # List imports
iS                      # List sections
px 64                   # Hex dump 64 bytes
s main                  # Seek to main
aaa                     # Full analysis
```

### Decompiler Usage

```bash
# Install rz-ghidra plugin
r2pm -ci rz-ghidra

# Access in Cutter
# Window → Decompiler (if installed)
# Or use Console: pdg @main
```

### Graph View Navigation

| Action | Shortcut |
|--------|----------|
| Zoom in/out | `Ctrl+Scroll` |
| Pan | `Click + Drag` |
| Follow edge | `Double-click edge` |
| Node context menu | `Right-click node` |
| Copy disassembly | `Right-click → Copy` |
| Add comment | `Right-click → Comment` |

### Batch Analysis

```bash
# Quick analysis with output
cutter -q -c 'aaa; afl; iz' /path/to/binary

# Analyze and save project
cutter -p project.rzdb /path/to/binary
```

### Configuration

```
# In Cutter console or via command line:
e scr.color=0              # Disable colors
e asm.arch=arm             # Set architecture
e asm.bits=32              # Set bit width
e asm.syntax=intel         # Intel syntax
e bin.cache=true           # Cache binary info
```

### Plugin Management

```bash
# Via Cutter's Rizin console:
r2pm -s <keyword>          # Search plugins
r2pm -ci <plugin>          # Install plugin
r2pm -l                    # List installed
r2pm -u <plugin>           # Uninstall plugin

# Recommended plugins:
r2pm -ci rz-ghidra         # Ghidra decompiler
r2pm -ci r2dec             # JavaScript decompiler
```

### Export Options

| Format | Method |
|--------|--------|
| CSV function list | Script via Console |
| JSON data | `aflj` in Console |
| Screenshot | File → Export |
| Patched binary | File → Save (after modifications) |

### Comparison with Alternatives

| Feature | Cutter | Ghidra | IDA Pro |
|---------|--------|--------|---------|
| Price | Free | Free | $$$$ |
| GUI | Yes | Yes | Yes |
| Decompiler | rz-ghidra | Built-in | Hex-Rays |
| Speed | Fast | Slow startup | Medium |
| Scripting | Rizin | Java/Python | IDAPython |
| Learning curve | Low | Medium | High |
