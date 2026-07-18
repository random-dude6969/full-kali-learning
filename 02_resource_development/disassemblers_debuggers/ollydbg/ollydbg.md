---
tool_name: "ollydbg"
display_name: "OllyDbg"
mitre_tactic: "resource_development"
mitre_tactic_id: "TA0042"
mitre_subcategory: "disassemblers_debuggers"
install_command: "sudo apt install ollydbg"
version: "1.10"
github_repo: ""
kali_tools_page: "https://www.kali.org/tools/ollydbg/"
difficulty_level: "beginner-to-intermediate"
requires_root: false
gui_available: true
tags: ["reverse-engineering", "debugger", "windows", "x86", "gui"]
related_tools: ["edb-debugger", "x64dbg", "gdb", "immunity-debugger"]
---

# OllyDbg

## Introduction & Overview

OllyDbg is a 32-bit assembler-level analyzing debugger for Microsoft Windows. Released in 2000 by Oleh Yuschuk, it became the most popular user-mode Windows debugger for over a decade. Its intuitive GUI, powerful breakpoint system, and built-in code analysis made it the go-to tool for Windows reverse engineering, exploit development, and malware analysis.

OllyDbg is not actively maintained — the last official release (v2.01) dates to 2013, and v1.10 (the version packaged in Kali) remains the most widely used. Despite its age, OllyDbg's design principles directly influenced modern successors like x64dbg and immunity-debugger.

### What OllyDbg Actually Does

OllyDbg attaches to Windows processes (or loads EXE files directly) and provides a multi-pane view: CPU disassembly, registers, memory dump, stack, and a log window. It supports hardware and software breakpoints, memory patching, tracing, and conditional breakpoint expressions written in a simple scripting language.

### When to Use OllyDbg

- Analyzing 32-bit Windows executables on Kali via Wine
- Learning x86 assembly and debugging fundamentals
- Quick one-off binary analysis of Windows PE files
- Exploit development targeting 32-bit Windows applications
- Understanding legacy malware that targets old Windows versions

### When NOT to Use OllyDbg

- Analyzing 64-bit Windows binaries (use x64dbg instead)
- Linux binary analysis (use GDB with GEF)
- When you need an actively maintained tool with modern features
- Complex malware analysis with anti-debugging (use IDA Pro or Ghidra)

### Legal and Ethical Considerations

OllyDbg runs under Wine on Linux systems. Analyzing software you own or have authorization to test is legal. Reverse engineering for interoperability, security research, and education is generally protected under the DMCA's reverse engineering exemptions. Never use OllyDbg to analyze software in ways that violate its license agreement or applicable laws without authorization.

### Key Concepts

- **User-Mode Debugging**: OllyDbg operates entirely in user mode — it cannot debug kernel-mode drivers or system processes.
- **Byte-Patching**: Modify running code in real-time by NOP-ing instructions or writing new bytes directly in the disassembly view.
- **Conditional Breakpoints**: Pause execution based on register values, memory contents, or complex expressions without writing scripts.
- **Tracing**: Record every instruction executed to build a complete execution history — invaluable for understanding obfuscated code.
- **Plugin Architecture**: OllyDbg supports plugins (OllyDbg PDK) for extending functionality, though the ecosystem is mostly frozen.

## Installation & Setup

### System Requirements

- Wine (32-bit support required)
- A Windows executable to analyze
- Minimum 512MB RAM (1GB recommended for complex binaries)

### Installation on Kali Linux

```bash
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt install wine32 ollydbg
```

**Explanation:** OllyDbg is a native 32-bit Windows executable. Wine32 provides the 32-bit Windows API translation layer needed to run it on Linux. The `--add-architecture i386` enables 32-bit package support.

### Launch OllyDbg

```bash
wine /usr/share/ollydbg/OLLYDBG.EXE
```

**Explanation:** Executes the OllyDbg Windows binary through Wine, which translates Windows API calls to Linux system calls in real time.

### Verification

After launch, OllyDbg should display its main window with the CPU view, register pane, and log window. The title bar should show "OllyDbg v1.10".

### Configuration Tips

- Set Wine to Windows 7 mode: `winecfg` → Windows Version → Windows 7
- Increase heap allocation if OllyDbg crashes on large binaries
- Create a Wine prefix dedicated to RE tools: `WINEPREFIX=~/.wine_re wine /usr/share/ollydbg/OLLYDBG.EXE`

## Basic Usage

### Opening a Binary

```bash
wine /usr/share/ollydbg/OLLYDBG.EXE target.exe
```

**Explanation:** Opens the specified Windows executable directly in the debugger. OllyDbg automatically analyzes the code, identifies functions, and labels known API calls.

### Key Keyboard Shortcuts

| Key | Function |
|---|---|
| F2 | Set/clear breakpoint at cursor |
| F7 | Step into (execute one instruction) |
| F8 | Step over (execute one instruction, skip calls) |
| F9 | Run until breakpoint or exception |
| F12 | Pause execution |
| Ctrl+F2 | Restart debugging session |
| Alt+F4 | Close OllyDbg |

### Setting Breakpoints

Click on the target instruction in the CPU disassembly window and press F2. A red highlight indicates an active breakpoint. Press F2 again to remove it.

### Examining Memory

1. In the CPU window, right-click → Search for → All referenced text strings
2. This reveals string references throughout the binary
3. Double-click any string to jump to the code that references it

### Patching Code

1. Navigate to the instruction to patch
2. Press Space to open the Assemble dialog
3. Type the new instruction (e.g., `NOP`)
4. Click Assemble
5. Right-click → Copy to executable → Save file

### Reading the Register Window

The register pane shows:
- **General Purpose**: EAX, EBX, ECX, EDX, ESI, EDI, EBP, ESP
- **EIP**: Current instruction pointer (where execution will resume)
- **Flags**: Zero Flag, Carry Flag, Sign Flag, etc.
- **Segments**: CS, DS, ES, FS, GS, SS

## Intermediate Usage

### Conditional Breakpoints

Instead of pausing on every hit, set conditions that only trigger the breakpoint when specific criteria are met.

Right-click a breakpoint → Condition → enter an expression:

```
[ESP+4] == 0x12345678
```

**Explanation:** This breakpoint only triggers when the value at ESP+4 equals `0x12345678` — useful for breaking only on specific function parameters.

### Tracing Execution

1. Set a starting breakpoint at the function entry
2. Run to the breakpoint (F9)
3. Press Ctrl+F11 (Trace Into) or Ctrl+F12 (Trace Over)
4. OllyDbg records every instruction executed
5. View the trace log in the Trace window

### Analyzing API Calls

OllyDbg automatically identifies Windows API calls and labels them. In the CPU window:

1. Right-click → Search for → All intermodular calls
2. This lists every imported function call in the binary
3. Double-click to navigate to each call site

### Working with Plugins

OllyDbg plugins are `.dll` files placed in the OllyDbg directory:

```
usr/share/ollydbg/plugins/
```

Notable plugins:
- **OllyDump**: Dump the process memory to reconstruct unpacked executables
- **Phantom**: Anti-anti-debugging plugin that hides the debugger from detection
- **HideOD**: Hides OllyDbg from debugger-detection routines

### Scripting with OllyDbg

OllyDbg's scripting language supports basic automation:

```
log "Function called with arg: {arg1}"
run
cmp eax, 0
je not_found
```

**Explanation:** OllyDbg scripts execute as breakpoints fire, enabling basic conditional logic and logging during debugging sessions.

## Advanced Usage

### Unpacking Malware with OllyDbg

Many malware samples use packers (UPX, Themida, ASProtect) to hide their true code. OllyDbg excels at unpacking:

1. Load the packed executable
2. Follow the execution past the unpacking stub
3. Set a breakpoint on `VirtualProtect` or `WriteProcessMemory` (common unpacking APIs)
4. Run until the breakpoint triggers
5. Dump the unpacked code from memory using OllyDump plugin
6. Fix the dumped PE header with a PE header fixer

### Anti-Debugging Bypass

Malware often detects debuggers. Common checks and OllyDbg countermeasures:

| Detection Method | Bypass |
|---|---|
| `IsDebuggerPresent()` | Patch `FS:[30]` (PEB.BeingDebugged) |
| `NtQueryInformationProcess` | Use HideOD plugin |
| Timing checks (RDTSC) | Patch timing comparison |
| Hardware breakpoint detection | Use software breakpoints only |

### OllyDbg vs Modern Alternatives

| Feature | OllyDbg v1.10 | x64dbg | Immunity Debugger |
|---|---|---|---|
| 32-bit support | Yes | Yes | Yes |
| 64-bit support | No | Yes | No |
| Actively maintained | No | Yes | Limited |
| Python scripting | No | Yes | Yes |
| Price | Free | Free | Free |
| Learning curve | Low | Medium | Medium |

## Real-World Scenarios

### Scenario 1: Analyzing a Suspicious Windows Binary

A malware sample arrives as an email attachment. Quick triage with OllyDbg:

1. Open the binary in OllyDbg
2. Check the Import tab — if it imports `URLDownloadToFile`, `CreateProcess`, and `RegSetValue`, it is likely a dropper
3. Run to the program entry (not `main` — the actual OEP)
4. Trace through the unpacking stub
5. Dump the unpacked binary for deeper analysis in Ghidra

### Scenario 2: CTF Challenge - Windows Crackme

A CTF challenge provides a Windows crackme that validates a license key:

1. Load the crackme in OllyDbg
2. Search for referenced text strings — find "Correct!" and "Wrong!" messages
3. Navigate to the code that references "Correct!"
4. Set a breakpoint on the conditional jump before it
5. Run with a test key, examine the comparison logic
6. Extract the validation algorithm or patch the jump to always succeed

### Scenario 3: Understanding Exploit Development

Study how buffer overflows work on Windows:

1. Compile a vulnerable program with `/GS` and `/SAFESEH` disabled
2. Load in OllyDbg
3. Send an oversized buffer to trigger the overflow
4. Observe EIP overwritten with your controlled value
5. Use OllyDbg's stack view to identify ESP location
6. Craft a payload with a JMP ESP gadget address

## Detection & Defense

### How Defenders Detect OllyDbg Usage

OllyDbg has well-known artifacts that malware checks for:

- **Window class name**: `"OllyDbg"` — malware enumerates window titles
- **Process name**: `OLLYDBG.EXE` — checked via `GetModuleFileName`
- **DLL artifacts**: The debugger injects DLLs that can be detected
- **PEB flag**: `BeingDebugged` in the Process Environment Block is set
- **Hardware breakpoints**: DR0-DR3 debug registers are non-zero

### Blue Team Response

- Monitor for Wine execution of OllyDbg in production environments
- Alert on processes running from `/usr/share/ollydbg/`
- Check for Wine prefix creation as an indicator of RE tooling setup
- Correlate OllyDbg usage with other reconnaissance indicators

## Troubleshooting

### Wine Configuration Issues

**Error: `Application could not be started`**

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32
wineboot --init
```

**Explanation:** OllyDbg requires the 32-bit Wine environment. The error occurs when only 64-bit Wine is installed.

### OllyDbg Crashes on Large Binaries

Increase Wine's heap allocation:

```bash
WINEDEBUG=-all wine /usr/share/ollydbg/OLLYDBG.EXE large_binary.exe
```

**Explanation:** Suppressing debug output reduces memory pressure. For persistent crashes, create a fresh Wine prefix.

### Display Issues Under Wine

If the GUI renders incorrectly:

```bash
winetricks gdiplus
```

**Explanation:** Installs the GDI+ library which OllyDbg uses for some rendering operations.

### FAQ

**Q: Should I use OllyDbg v1.10 or v2.01?**
A: Use v1.10 (the Kali version). It is more stable, better documented, and has more plugins. v2.01 is alpha-quality and not recommended.

**Q: Can OllyDbg debug .NET applications?**
A: No. Use dnSpy or ILSpy for .NET reverse engineering. OllyDbg only handles native x86 code.

**Q: Why not just use x64dbg instead?**
A: For new work, use x64dbg. But OllyDbg remains useful for quick analysis of 32-bit binaries, and its simplicity makes it ideal for learning debugging fundamentals.

## Resources

### Official Documentation
- [OllyDbg Homepage](http://www.ollydbg.de/)
- [OllyDbg v1.10 Help File](http://www.ollydbg.de/odbg.htm)

### Tutorials and Guides
- [OllyDbg Tutorial by 0xDeviL](https://www.youtube.com/watch?v=OllyDbg_tutorial)
- [Practical Malware Analysis — Chapter 3: OllyDbg](https://practicalmalwareanalysis.com/)
- [OllyDbg for Absolute Beginners](https://beginners.re/)

### Books
- *Reverse Engineering for Beginners* by Dennis Yurichev (free, uses OllyDbg examples)
- *Practical Malware Analysis* by Sikorski and Honig
- *The IDA Pro Book* by Chris Eagle

### Related Tools
- [edb-debugger](../edb-debugger/edb-debugger.md) — Linux equivalent of OllyDbg
- [x64dbg](https://x64dbg.com/) — Modern successor, supports 32-bit and 64-bit
- [Immunity Debugger](https://www.immunityinc.com/products/debugger/) — Python-scriptable Windows debugger
- [GDB](../gdb/gdb.md) — Linux debugging alternative
