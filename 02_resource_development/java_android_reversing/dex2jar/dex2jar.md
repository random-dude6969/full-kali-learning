---
tool_name: dex2jar
display_name: dex2jar - DEX to JAR Converter
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: java_android_reversing
install_command: "sudo apt install dex2jar"
version: "2.4"
github_repo: "https://github.com/pxb1988/dex2jar"
kali_tools_page: "https://www.kali.org/tools/dex2jar/"
difficulty_level: beginner
requires_root: false
gui_available: false
tags: [android, dex, jar, converter, bytecode, smali]
related_tools: [jadx, apktool, jadx-gui, jd-gui]
---

# dex2jar - DEX to JAR Converter

## 1. Introduction & Overview

dex2jar is a toolset for converting Android DEX (Dalvik Executable) files into standard Java JAR files. This conversion enables the use of standard Java analysis tools (JD-GUI, Procyon, etc.) on Android applications. The project also includes smali/baksmali for DEX disassembly/assembly.

**Created:** 2012 by Panxiaobo (pxb1988)
**License:** Apache 2.0
**GitHub Stars:** 13,100+

### What It Does

- Converts `.dex` files to `.class` files (packaged as `.jar`)
- Converts entire `.apk` files (extracts DEX and converts)
- Provides smali disassembler (DEX → smali text)
- Provides smali assembler (smali text → DEX)
- Includes d2j-decrypt-string for string decryption

### DEX vs JAR Format

| Aspect | DEX (Dalvik) | JAR (Java) |
|--------|-------------|------------|
| **VM** | Dalvik/ART | JVM |
| **Bytecode** | Dalvik opcodes | Java opcodes |
| **Register-based** | Yes | No (stack-based) |
| **Multi-DEX** | Yes (classes.dex, classes2.dex) | Single classpath |
| **Tool support** | Limited | Extensive |

### When to Use

- **Bridge to Java tools**: Convert DEX to JAR for analysis with JD-GUI, IDA, etc.
- **Smali workflow**: Use smali/baksmali for assembly-level DEX editing
- **Legacy tool compatibility**: When newer tools like JADX aren't available
- **String decryption**: Use d2j-decrypt-string for encrypted Android strings

### When NOT to Use

- JADX can decompile directly from APK/DEX (preferred for source code)
- You need to rebuild modified APKs (use Apktool)
- iOS or desktop Java apps (use JD-GUI directly)

### Legal & Ethical

dex2jar is for authorized security research and education. Converting and analyzing commercial applications may require authorization.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **DEX** | Dalvik Executable — Android's bytecode format |
| **Smali** | Human-readable Dalvik assembly language |
| **BakSmali** | Disassembler (DEX → smali) |
| **Smali** (tool) | Assembler (smali → DEX) |
| **d2j-dex2jar** | Main conversion tool |

## 2. Installation & Setup

### System Requirements

- Java 8+ (JRE)
- ~5 MB disk space

### Installation

```bash
sudo apt install dex2jar
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  dex2jar
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 3.2 MB of archives.
After this operation, 12.8 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 dex2jar all 2.4-0kali1 [3.2 MB]
Fetched 3.2 MB in 0s (16.2 MB/s)
Selecting previously unselected package dex2jar.
Preparing to unpack .../dex2jar_2.4-0kali1_all.deb ...
Unpacking dex2jar (2.4-0kali1) ...
Setting up dex2jar (2.4-0kali1) ...
```

### From GitHub Releases (Alternative)

```bash
wget https://github.com/pxb1988/dex2jar/releases/download/v2.4/dex-tools-v2.4.zip
unzip dex-tools-v2.4.zip -d dex2jar
chmod +x dex2jar/d2j-dex2jar.sh
```

### Verification

```bash
d2j-dex2jar.sh --version 2>/dev/null || ls /usr/share/dex2jar/
```

## 3. Basic Usage

### Converting a DEX File to JAR

```bash
d2j-dex2jar.sh classes.dex
```

**Explanation:** Converts `classes.dex` to `classes-dex2jar.jar`.

**Example output:**
```
dex2jar classes.dex -> classes-dex2jar.jar
```

### Converting an APK File

```bash
d2j-dex2jar.sh -f target_app.apk
```

**Explanation:** Extracts DEX files from the APK and converts them to a JAR. The `-f` flag forces overwrite.

**Example output:**
```
dex2jar target_app.apk -> target_app-dex2jar.jar
```

### Converting with Output Path

```bash
d2j-dex2jar.sh classes.dex -o output.jar
```

**Explanation:** Specifies the output JAR file path explicitly.

### Decompiling the Result

```bash
# After conversion, use JD-GUI or JADX on the JAR
jd-gui target_app-dex2jar.jar
# Or:
jadx -d output_dir target_app-dex2jar.jar
```

**Explanation:** The converted JAR can be opened with any standard Java decompiler.

### Flags Reference

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force overwrite existing output |
| `-o FILE`, `--output FILE` | Output JAR file path |
| `-r`, `--no-repackage` | Don't repackage to standard Java package names |
| `--skip-exceptions` | Skip exception handling decompilation |
| `--debug` | Debug output |

## 4. Intermediate Usage

### Handling Multi-DEX APKs

```bash
# Some APKs have multiple DEX files
unzip -l target.apk | grep "\.dex"
# classes.dex, classes2.dex, classes3.dex

# Convert each
for dex in classes*.dex; do
    d2j-dex2jar.sh "$dex" -o "${dex%.dex}-jar"
done

# Merge JARs (if needed)
jar uf merged.jar -C classes-dex2jar .
jar uf merged.jar -C classes2-dex2jar .
```

**Explanation:** Multi-DEX apps need each DEX converted separately, then merged for analysis.

### Smali Disassembly

```bash
d2j-dex2jar.sh --smali classes.dex -o smali_output/
```

**Explanation:** Disassembles DEX to smali text files for reading and editing.

### Smali Assembly

```bash
d2j-smali.sh smali_output/ -o output.dex
```

**Explanation:** Assembles smali text files back to DEX format after editing.

### Batch Conversion

```bash
#!/bin/bash
for apk in *.apk; do
    name=$(basename "$apk" .apk)
    d2j-dex2jar.sh -f "$apk" -o "jars/${name}.jar"
    echo "Converted: $apk -> jars/${name}.jar"
done
```

**Explanation:** Automates conversion of multiple APK files.

### String Decryption

```bash
d2j-decrypt-string encrypted_strings.dex
```

**Explanation:** Decrypts obfuscated string constants in DEX files.

## 5. Advanced Usage

### Integration in Analysis Pipeline

```bash
#!/bin/bash
APK=$1
WORKDIR=$(mktemp -d)

# Step 1: Convert to JAR
d2j-dex2jar.sh -f "$APK" -o "$WORKDIR/classes.jar"

# Step 2: Convert to JAR for JD-GUI analysis
jd-gui -export "$WORKDIR/classes.jar" "$WORKDIR/decompiled/"

# Step 3: Search for interesting patterns
grep -r "http://\|https://" "$WORKDIR/decompiled/" | head -20
grep -r "password\|secret\|apikey" "$WORKDIR/decompiled/" | head -20

# Step 4: Cleanup
# rm -rf "$WORKDIR"
echo "Analysis saved to: $WORKDIR"
```

### Custom Smali Patches

```bash
# Disassemble
d2j-dex2jar.sh --smali classes.dex -o smali_dir/

# Edit smali files
vim smali_dir/com/example/app/MainActivity.smali

# Reassemble
d2j-smali.sh smali_dir/ -o patched.dex

# Repackage
zip -j modified.apk patched.dex
```

**Explanation:** Disassemble, edit smali, reassemble — the classic Android patching workflow.

### Comparing DEX Versions

```bash
# Convert both versions
d2j-dex2jar.sh -f old_version.apk -o old.jar
d2j-dex2jar.sh -f new_version.apk -o new.jar

# Diff decompiled output
jadx -d old_src/ old.jar
jadx -d new_src/ new.jar
diff -r old_src/ new_src/
```

## 6. Real-World Scenarios

### Scenario 1: Quick APK Analysis

**Target:** Unknown APK file

```bash
# Convert
d2j-dex2jar.sh -f unknown.apk

# Open in JD-GUI
jd-gui unknown-dex2jar.jar

# Look for main activity, network calls, data handling
```

### Scenario 2: Malware String Extraction

```bash
d2j-dex2jar.sh -f malware.apk -o malware.jar
jadx -d malware_analysis/ malware.jar
grep -roh "http[s]*://[^\"']*" malware_analysis/ | sort -u
```

### Scenario 3: CTF Android Challenge

```bash
d2j-dex2jar.sh -f challenge.apk
jadx -d challenge_src/ challenge-dex2jar.jar
grep -rn "flag\|FLAG\|secret" challenge_src/
```

### Scenario 4: App Comparison

```bash
# Compare two versions
d2j-dex2jar.sh -f v1.apk -o v1.jar
d2j-dex2jar.sh -f v2.apk -o v2.jar
jadx -d v1/ v1.jar
jadx -d v2/ v2.jar
diff <(find v1/sources -name "*.java" | sort) <(find v2/sources -name "*.java" | sort)
```

## 7. Detection & Defense

### How Defenders Detect

- **Signature verification**: Modified APKs fail signature checks
- **Runtime integrity**: Apps can verify their own code at runtime
- **Obfuscation**: ProGuard/R8 makes decompiled code harder to interpret

### Mitigation

- Use code obfuscation and string encryption
- Implement runtime integrity verification
- Use Android App Bundle for dynamic delivery

### Blue Team Response

```bash
# Convert and analyze suspicious APK
d2j-dex2jar.sh -f suspicious.apk
jadx -d analysis/ suspicious-dex2jar.jar
grep -r "Runtime.exec\|ProcessBuilder" analysis/
```

## 8. Troubleshooting

### Common Errors

**"Can't read dex file"**
**Fix:** Verify the file is a valid DEX: `file classes.dex` (should say "Dalvik dex file").

**"Unsupported class version"**
**Fix:** Update dex2jar to the latest version or use JADX instead.

**"conversion failed"**
Some heavily obfuscated DEX files may fail conversion. Try using JADX which handles more DEX variants.

**"command not found: d2j-dex2jar.sh"**
**Fix:** `sudo apt install dex2jar` or download from GitHub releases.

### Performance Optimization

- Use `-f` flag to avoid prompts about overwriting
- For large APKs, increase Java heap: `JAVA_OPTS="-Xmx2g" d2j-dex2jar.sh large.apk`

### FAQ

**Q: dex2jar vs JADX?**
A: dex2jar converts DEX to JAR (you then need a separate decompiler). JADX decompiles DEX directly to Java source. JADX is more convenient; dex2jar is useful for legacy workflows.

**Q: Is dex2jar maintained?**
A: The repo has limited recent activity (last release: 2023). Consider JADX for new projects.

**Q: Can I use the output JAR with Procyon/CFR?**
A: Yes. The JAR is standard Java bytecode — any Java decompiler can read it.

---

## Resources

- **GitHub**: https://github.com/pxb1988/dex2jar
- **Releases**: https://github.com/pxb1988/dex2jar/releases
- **Smali Documentation**: https://github.com/JesusFreke/smali
- **Android DEX Format**: https://source.android.com/docs/core/runtime/dex-format
