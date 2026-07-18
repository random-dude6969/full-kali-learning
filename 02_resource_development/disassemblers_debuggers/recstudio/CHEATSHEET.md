# RE Studio (RecStudio) Cheatsheet

## Quick Reference Card

### Installation

```bash
sudo apt install recstudio
```

### Launch

```bash
recstudio                              # Start GUI
recstudio /path/to/binary              # Open binary directly
```

### Interface Panels

| Panel | Purpose |
|-------|---------|
| Disassembly | Disassembled instructions with addresses |
| Function List | All identified functions |
| String Table | Extracted strings from binary |
| Import Table | External library dependencies |
| Export Table | Exported symbols |
| Hex Dump | Raw binary data view |

### Navigation

| Action | How |
|--------|-----|
| Go to address | Press `G` and enter address |
| Cross-references | Press `X` at any reference |
| Rename symbol | Press `N` at function/label |
| Toggle hex view | Press `Space` |
| Search | Press `Ctrl+F` |
| Back | `Alt+Left` |
| Forward | `Alt+Right` |

### Analysis Workflow

```
1. File → Open → Select binary
2. Auto-analysis identifies functions
3. Browse Function List for entry points
4. Click function to view disassembly
5. Examine Import Table for capabilities
6. Extract strings for indicators
7. Add comments/labels as needed
8. File → Save to preserve analysis
```

### String Analysis

| Action | How |
|--------|-----|
| View all strings | Open String Table panel |
| Search strings | Type in search bar |
| Filter by type | Select ASCII/Unicode filter |
| Jump to reference | Double-click string entry |

### Import Analysis

| Import | Potential Use |
|--------|---------------|
| `URLDownloadToFile` | Downloading payloads |
| `CreateProcess` | Executing processes |
| `RegSetValueEx` | Registry modification |
| `InternetOpenA` | Network communication |
| `CryptEncrypt` | Data encryption |
| `socket`/`connect` | Network connections |
| `gets`/`strcpy` | Potential vulnerabilities |

### Cross-Reference Analysis

| Action | How |
|--------|-----|
| View xrefs to address | Right-click → Cross-references |
| Follow call chain | Click function call target |
| Navigate back | Alt+Left |

### Key Differences from Other Tools

| Feature | RecStudio | Ghidra | Cutter |
|---------|-----------|--------|--------|
| Weight | Light | Heavy | Medium |
| Decompiler | No | Yes | rz-ghidra |
| Learning curve | Low | Medium | Low |
| Startup speed | Fast | Slow | Fast |
| Binary formats | PE, ELF | Many | Many |

### Tips

- Use RecStudio for quick triage, not deep analysis
- Export strings to file for grep-based analysis
- Combine with command-line tools: `strings`, `file`, `objdump`
- Check the Import table first — it reveals the binary's capabilities
- For packed binaries, extract with UPX first: `upx -d packed.exe`
- Use RecStudio alongside GDB for combined static/dynamic analysis
