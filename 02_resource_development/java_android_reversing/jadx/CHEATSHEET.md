# JADX Cheatsheet

## Quick Commands

```bash
jadx -d output/ target.apk                    # Decompile APK
jadx --deobf -d output/ target.apk            # With deobfuscation
jadx-gui target.apk                           # Launch GUI
jadx --version                                # Show version
```

## Basic Decompilation

```bash
jadx -d output/ target.apk                    # Full decompile
jadx -d output/ classes.dex                   # DEX file
jadx -d output/ target.apk --no-res           # Source only
jadx -d output/ target.apk --no-src           # Resources only
```

## Deobfuscation

```bash
jadx --deobf -d output/ target.apk            # Enable deobfuscation
jadx --deobf --deobf-min 4 --deobf-max 100 -d output/ target.apk
```

## Export Formats

```bash
jadx -d output/ --export-gradle target.apk    # As Gradle project
jadx -d output/ --output-format json target.apk  # JSON output
```

## Single Class

```bash
jadx -d output/ --single-class com.example.app.MainActivity target.apk
```

## Analysis

```bash
jadx -d output/ --call-graph dot target.apk   # Call graph
jadx -d output/ --cfg target.apk              # Control flow graphs
```

## Flags

| Flag | Description |
|------|-------------|
| `-d DIR` | Output directory |
| `-r`, `--no-res` | Skip resources |
| `-s`, `--no-src` | Skip source |
| `-j NUM` | Thread count (default: 16) |
| `-e`, `--export-gradle` | Gradle export |
| `--deobf` | Enable deobfuscation |
| `--deobf-min NUM` | Min name length (3) |
| `--deobf-max NUM` | Max name length (64) |
| `--show-bad-code` | Show corrupted code |
| `--single-class CLASS` | Single class only |
| `--call-graph` | Save call graph |
| `--cfg` | Save CFGs |
| `-f`, `--fallback` | Fallback mode |
| `-q`, `--quiet` | Quiet output |
| `-v`, `--verbose` | Verbose output |

## Security Audit Commands

```bash
# After decompilation
grep -rn "http://" output/sources/              # Insecure HTTP
grep -rn "password\|secret\|api_key" output/sources/
grep -rn "getDeviceId\|getSubscriberId" output/sources/
grep -rn "Cipher\|encrypt\|decrypt" output/sources/
grep -rn "DexClassLoader\|loadClass" output/sources/
grep -rn "TrustManager" output/sources/
```

## Environment Variables

```bash
JADX_DISABLE_XML_SECURITY=true       # Disable XML checks
JADX_ZIP_MAX_ENTRIES_COUNT=200000    # Max zip entries
JADX_CONFIG_DIR=/path/to/config      # Custom config
```

## Batch Processing

```bash
for apk in *.apk; do
    jadx -d "output/$(basename $apk .apk)" --deobf "$apk"
done
```

## GUI Features

- Syntax-highlighted source code
- Jump to declaration
- Find usage
- Full text search
- Smali debugger

## Tips

- Use `--deobf` for obfuscated apps
- Use `--no-res` for faster source-only analysis
- Use `--show-bad-code` when decompilation fails
- Increase `-j` for faster processing
- Use jadx-gui for interactive exploration
