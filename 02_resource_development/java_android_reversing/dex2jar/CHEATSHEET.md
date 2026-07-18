# dex2jar Cheatsheet

## Quick Commands

```bash
d2j-dex2jar.sh classes.dex                    # DEX → JAR
d2j-dex2jar.sh -f target.apk                  # APK → JAR
d2j-dex2jar.sh classes.dex -o output.jar      # Custom output
d2j-dex2jar.sh --smali classes.dex -o smali/  # DEX → smali
d2j-smali.sh smali/ -o output.dex             # smali → DEX
```

## Convert APK to JAR

```bash
d2j-dex2jar.sh -f target.apk
# Output: target-dex2jar.jar
```

## Convert DEX to JAR

```bash
d2j-dex2jar.sh classes.dex
# Output: classes-dex2jar.jar
```

## Smali Workflow

```bash
# Disassemble DEX to smali
d2j-dex2jar.sh --smali classes.dex -o smali_dir/

# Edit smali files
vim smali_dir/com/example/MyClass.smali

# Assemble back to DEX
d2j-smali.sh smali_dir/ -o patched.dex
```

## Flags

| Flag | Description |
|------|-------------|
| `-f`, `--force` | Force overwrite |
| `-o FILE` | Output file |
| `-r`, `--no-repackage` | Don't repackage |
| `--skip-exceptions` | Skip exception handling |
| `--debug` | Debug output |
| `--smali` | Output smali format |

## Multi-DEX Handling

```bash
# Check for multiple DEX
unzip -l target.apk | grep "\.dex"

# Convert each
for dex in classes*.dex; do
    d2j-dex2jar.sh "$dex" -o "${dex%.dex}.jar"
done
```

## Integration with Other Tools

```bash
# dex2jar → JD-GUI
d2j-dex2jar.sh target.apk
jd-gui target-dex2jar.jar

# dex2jar → JADX
d2j-dex2jar.sh target.apk
jadx -d output/ target-dex2jar.jar

# Batch conversion
for apk in *.apk; do
    d2j-dex2jar.sh -f "$apk" -o "jars/$(basename $apk .apk).jar"
done
```

## Quick Reference

| Tool | Command | Purpose |
|------|---------|---------|
| d2j-dex2jar | `.dex` → `.jar` | Format conversion |
| d2j-smali | smali → `.dex` | Assembly |
| d2j-decrypt-string | Decrypt | String decryption |

## Tips

- Use `-f` to avoid overwrite prompts
- Use jadx directly for better results on modern APKs
- Handle multi-DEX with a loop script
- Reassemble smali edits with d2j-smali.sh
- Increase heap for large files: `JAVA_OPTS="-Xmx2g"`
