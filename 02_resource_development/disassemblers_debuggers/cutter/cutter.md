---
tool_name: "cutter"
display_name: "Cutter"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install cutter-re"
version: "2.5.0"
github_repo: "https://github.com/rizinorg/cutter"
kali_tools_page: "https://www.kali.org/tools/cutter/"
difficulty_level: "beginner-to-advanced"
requires_root: false
gui_available: true
tags: ["reverse-engineering", "disassembly", "gui", "radare2", "rizin"]
related_tools: ["radare2", "rizin", "ghidra", "ida-pro"]
---

# Cutter

## Introduction & Overview

Cutter is a free and open-source reverse engineering platform powered by Rizin. It provides a polished graphical interface for binary analysis, disassembly, debugging, and decompilation — making the power of the Rizin command-line framework accessible through a visual workspace. Created by reverse engineers for reverse engineers, Cutter bridges the gap between raw CLI tools and commercial RE suites like IDA Pro.

### What Cutter Actually Does

Cutter wraps Rizin's entire analysis engine in a Qt-based GUI. It displays disassembly views, function graphs, hex dumps, string tables, import/export lists, and a built-in debugger. The rz-ghidra plugin integrates Ghidra's decompiler directly into Cutter, giving users pseudocode output without leaving the application.

### When to Use Cutter

- When you want Rizin's analysis power without memorizing commands
- Quick binary triage with visual function graphs
- Learning reverse engineering with an intuitive interface
- Collaborative analysis sessions where visual aids help
- Debugging binaries with a visual breakpoint and register interface

### When NOT to Use Cutter

- Headless or automated analysis pipelines (use Rizin directly)
- Deep malware analysis requiring scripting automation (use Ghidra)
- When IDA Pro's superior analysis is justified by project scope
- Analyzing extremely large binaries (>500MB) where GUI lag becomes problematic

### Legal and Ethical Considerations

Cutter is a passive analysis tool. Using it to analyze software you own, have authorization to test, or are researching for legitimate security purposes is legal. The tool itself does not exploit vulnerabilities or interact with remote systems. Reverse engineering for interoperability, security research, and education is generally protected under DMCA and EUCD exemptions.

### Key Concepts

- **Rizin Backend**: All analysis is performed by Rizin — Cutter is the visual frontend.
- **Decompilation**: Via the rz-ghidra plugin, Cutter can show Ghidra's decompiler output.
- **Graph View**: Visual control flow graph showing function basic blocks and edges.
- **Debugging**: Built-in debugger supports breakpoints, stepping, and memory inspection.
- **Plugins**: Cutter supports Python and C++ plugins for extensibility.

## Installation & Setup

### System Requirements

- Linux, macOS, or Windows
- Qt 5.15+ or Qt 6.x
- Rizin library (automatically bundled in releases)
- 2GB RAM minimum, 4GB recommended
- Display: 1280x720 minimum

### Installation via Package Manager

```bash
sudo apt install cutter-re
```

**Explanation:** Installs Cutter from the Kali Linux repositories, including Rizin and all required dependencies.

### AppImage (Recommended for Latest Version)

```bash
wget https://github.com/rizinorg/cutter/releases/latest/download/Cutter-x86_64.AppImage
chmod +x Cutter-x86_64.AppImage
./Cutter-x86_64.AppImage
```

**Explanation:** The AppImage is self-contained and includes the latest Cutter build with all dependencies. No system-wide installation required.

### Build from Source

```bash
git clone --recurse-submodules https://github.com/rizinorg/cutter.git
cd cutter
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

**Explanation:** Builds Cutter from source with the Rizin submodule. Requires Qt5/Qt6, CMake, and a C++17 compiler.

### Docker

```bash
docker pull rizinorg/cutter
docker run -it --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix rizinorg/cutter
```

**Explanation:** Runs Cutter in a Docker container with X11 forwarding for the GUI. Useful for isolated analysis environments.

### Install rz-ghidra Plugin (Decompiler)

```bash
r2pm -ci rz-ghidra
```

**Explanation:** Installs the Ghidra decompiler plugin for Rizin, which Cutter uses to display pseudocode output. This step is optional but highly recommended.

### Verify Installation

```bash
cutter --version
```

**Explanation:** Displays the Cutter version, confirming successful installation.

## Basic Usage

### Opening a Binary

```bash
cutter /usr/bin/ls
```

**Explanation:** Opens the `ls` binary in Cutter's analysis interface, automatically loading functions, strings, and imports.

### The Analysis Dialog

On first load, Cutter presents an analysis dialog:

- **Analysis Depth**: Choose between quick scan (fast) and deep analysis (thorough)
- **Slider**: Move between minimal and comprehensive analysis
- **Default**: Use the middle setting for balanced speed and detail

### Navigating the Interface

The main interface has these key panels:

| Panel | Purpose |
|---|---|
| Disassembly | View disassembled instructions |
| Graph | Visual control flow graph |
| Hexdump | Raw binary data view |
| Strings | Extracted strings table |
| Functions | List of all identified functions |
| Imports/Exports | External dependencies and exports |
| Decompiler | Pseudocode output (rz-ghidra) |

### Searching for Strings

1. Open the Strings panel from the sidebar
2. Type in the search bar to filter
3. Double-click any string to jump to its reference in the code

### Function Navigation

1. Open the Functions panel
2. Click any function to jump to its disassembly
3. Use the Back/Forward buttons (or Alt+Left/Right) to navigate history

### Setting Breakpoints

1. Navigate to the target instruction in the Disassembly view
2. Press F2 or right-click → Toggle Breakpoint
3. The instruction turns red to indicate an active breakpoint
4. Start debugging with Debug → Start Debugging

## Intermediate Usage

### Graph View Analysis

The graph view shows function basic blocks as nodes connected by edges:

- **Green edges**: True branch (conditional jumps)
- **Red edges**: False branch
- **Gray edges**: Unconditional jumps/calls

Right-click a node to:
- Copy the disassembly text
- Add a comment
- Create a function at that address
- Follow jumps to other functions

### Customizing the Display

Cutter offers extensive display customization:

```bash
cutter -e scr.color=0 /usr/bin/ls
```

**Explanation:** Disables color output from the command line, useful for scripting or clean output.

### Decompiler View

With rz-ghidra installed:

1. Open any function in the disassembly view
2. The Decompiler panel shows the equivalent C pseudocode
3. Click on pseudocode lines to highlight the corresponding assembly
4. Rename variables and functions for clarity

### Using Rizin Commands Inside Cutter

Cutter has an integrated console for Rizin commands:

```bash
# In Cutter's console:
afl          # List all functions
axt sym.main # Cross-references to main
iz           # List strings
ii           # List imports
```

**Explanation:** The integrated console gives direct access to Rizin's command set without leaving the Cutter interface.

### Bookmarks and Comments

1. Right-click any instruction → Add Bookmark
2. View all bookmarks in the Bookmarks panel
3. Add comments via right-click → Add Comment
4. Comments appear as inline annotations in the disassembly

### Renaming and Retyping

Cutter allows renaming functions and variables:

1. Right-click a function name → Rename
2. In the decompiler view, right-click a variable → Change Type
3. Set the type to `int`, `char*`, `FILE*`, or custom structs
4. Renames propagate through cross-references

## Advanced Usage

### Custom Rizin Scripting in Cutter

Load a Rizin script through Cutter's console or command line:

```bash
cutter -i analysis_script.rz /path/to/binary
```

**Explanation:** Executes a Rizin script at load time, applying custom analysis before presenting the GUI.

### Plugin Development

Create a Cutter plugin directory:

```bash
mkdir -p ~/.local/share/cutter/plugins/my_plugin/
cat > ~/.local/share/cutter/plugins/my_plugin/plugin.py << 'EOF'
from cutter import CutterDockWidget

class MyPlugin(CutterDockWidget):
    def __init__(self, core, parent):
        super().__init__("My Plugin", core, parent)
        # Plugin implementation here
        pass
EOF
```

**Explanation:** Cutter loads Python plugins from the plugin directory at startup. Each plugin can add panels, menus, and analysis features.

### Integration with Other Tools

Cutter works well alongside other RE tools:

```bash
# Export function list from Cutter's Rizin backend
rizin -q -c 'aflj' /usr/bin/ls > functions.json

# Use Cutter for visual analysis, Rizin for scripting
cutter suspicious.exe &
rizin -q -c 'aaa; pdf @main' suspicious.exe
```

**Explanation:** Cutter and Rizin share the same analysis database, so findings in one are available in the other.

### Batch Analysis

For analyzing multiple binaries:

```bash
for binary in /path/to/samples/*; do
    cutter -q -e analysis.noncode=true -c 'aaa; afl; q' "$binary" > "$(basename $binary).analysis"
done
```

**Explanation:** Runs Cutter in quiet mode for automated batch analysis of multiple binaries, generating function lists for each.

### Binary Patching

1. Open the binary in write mode: `cutter -w binary.exe`
2. Navigate to the target instruction
3. Right-click → Assemble to enter new assembly
4. Save with File → Save Project

## Real-World Scenarios

### Scenario 1: Malware Analysis Workshop

A team of analysts needs to collaboratively analyze a malware sample:

1. Open the sample in Cutter
2. Generate a function graph overview (screenshot the graph view)
3. Use the decompiler to understand key functions
4. Add comments documenting findings
5. Export the project file for team sharing

### Scenario 2: CTF Reverse Engineering Challenge

A CTF provides a binary that prints "Access Denied" without the correct input:

1. Open in Cutter
2. Search strings for "Access Denied" and "Access Granted"
3. Follow cross-references to find the comparison logic
4. Use the decompiler to understand the validation function
5. Extract the flag or patch the conditional jump

### Scenario 3: Firmware Analysis

An embedded device firmware image needs analysis:

```bash
# Identify the architecture
file firmware.bin

# Open in Cutter with the correct architecture
cutter -a arm -b 32 firmware.bin
```

**Explanation:** Cutter supports specifying the target architecture for firmware images that lack standard binary headers.

### Scenario 4: Vulnerability Research

1. Open the target binary in Cutter
2. Identify functions with user-controlled input (check imports for `gets`, `scanf`, `strcpy`)
3. Enable the decompiler view for each suspect function
4. Trace data flow from input to dangerous sinks
5. Document potential buffer overflow or injection points

## Detection & Defense

### How Defenders Detect Cutter

- Process monitoring: `cutter` or `rizin` processes visible in process listings
- Qt library loading: Qt5/Qt6 libraries loaded by an analysis process
- Rizin artifacts: Shared libraries from librizin loaded into memory
- File access patterns: Non-sequential binary file reads typical of disassembly
- Network indicators: Cutter plugins may connect to external services

### Blue Team Response

- Monitor for RE tool installation on non-development workstations
- Alert on rizin/cutter process execution in sensitive environments
- Check whether the activity corresponds to authorized security testing
- Review file integrity monitoring alerts for the analyzed binaries

## Troubleshooting

### Common Errors and Solutions

**Error: `cutter: command not found`**

```bash
sudo apt install cutter-re
```

**Explanation:** The package name differs from the command. Install `cutter-re` on Kali.

**Error: No decompiler output**

```bash
r2pm -ci rz-ghidra
```

**Explanation:** The Ghidra decompiler plugin is not installed by default. Install it with Rizin's package manager.

**Error: Qt version mismatch**

```bash
sudo apt install qtbase5-dev qttools5-dev
```

**Explanation:** Cutter requires specific Qt development libraries. Install the correct version for your system.

**Error: Cutter crashes on large binaries**

Increase the analysis timeout or reduce analysis depth:

```bash
cutter -e analysis.timeout=60 -A 0 /path/to/large_binary
```

**Explanation:** Large binaries may exceed the default analysis timeout. Setting a longer timeout or reducing analysis depth prevents crashes.

### Performance Optimization

- Disable unnecessary panels (View → Panels) to reduce rendering overhead
- Use the quick analysis slider for initial triage before deep analysis
- Close other applications when analyzing large binaries
- Use SSD storage for the Cutter working directory

### FAQ

**Q: Cutter vs Ghidra — which is better?**
A: Ghidra wins on analysis depth, decompiler quality, and scripting flexibility. Cutter wins on startup speed, ease of use, and Rizin ecosystem integration. For quick analysis, Cutter. For deep dives, Ghidra.

**Q: Can Cutter debug binaries remotely?**
A: Yes, through Rizin's remote debugging capabilities. Use `cutter -D gdb://target:port` to connect to a remote GDB stub.

**Q: Does Cutter support ARM64 binaries?**
A: Yes. Cutter inherits Rizin's multi-architecture support, covering x86, x64, ARM, ARM64, MIPS, PPC, RISC-V, and more.

## Resources

### Official Documentation
- [Cutter Documentation](https://cutter.re/docs/)
- [Cutter GitHub](https://github.com/rizinorg/cutter)
- [Rizin Book](https://book.rada.re/)

### Tutorials and Guides
- [Cutter Getting Started Guide](https://cutter.re/docs/user-docs.html)
- [Cutter Plugin Development](https://cutter.re/docs/plugins.html)
- [Rizin Command Cheat Sheet](https://book.rada.re/rizin/)

### Books
- *The IDA Pro Book* by Chris Eagle (RE concepts applicable to Cutter)
- *Practical Binary Analysis* by Dennis Andriesse
- *Reverse Engineering for Beginners* by Dennis Yurichev

### Videos
- [Cutter YouTube Tutorials](https://www.youtube.com/c/r2con)
- [RizinCon Conference Talks](https://www.youtube.com/c/r2con)

### Related Tools
- [Rizin](../rizin/rizin.md) — Cutter's command-line backend
- [Radare2](../radare2/radare2.md) — Rizin's predecessor
- [Ghidra](../ghidra/ghidra.md) — NSA's RE suite with decompiler
- [edb-debugger](../edb-debugger/edb-debugger.md) — Linux debugger alternative
