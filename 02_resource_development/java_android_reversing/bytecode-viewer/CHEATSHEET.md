# Bytecode Viewer Cheatsheet

## Quick Start

```bash
bytecode-viewer                                    # Launch GUI
java -jar bytecode-viewer.jar -help                # CLI help
java -jar bytecode-viewer.jar -list                # List decompilers
```

## CLI Usage

```bash
# Decompile with specific decompiler
java -jar BCV.jar -decompiler procyon -i target.jar -o output.java

# Decompile all classes
java -jar BCV.jar -i target.jar -o output/ -t all -decompiler cfr -nowait

# Decompile specific class
java -jar BCV.jar -i target.jar -o output.java -t "com.example.Main" -decompiler procyon
```

## CLI Flags

| Flag | Description |
|------|-------------|
| `-help` | Help menu |
| `-clean` | Delete BCV directory |
| `-english` | Force English UI |
| `-list` | List decompilers |
| `-decompiler NAME` | Select decompiler |
| `-i FILE` | Input file |
| `-o FILE` | Output file |
| `-t CLASS` | Target class |
| `-nowait` | No user prompt |

## Built-in Decompilers

| Decompiler | Best For |
|------------|----------|
| CFR | Modern Java, annotations |
| Procyon | Java 8+, accuracy |
| FernFlower | Readable output |
| JADX | Android APK |
| JD-GUI | Fast, reliable |
| Krakatau | Edge cases |

## File Formats

| Format | Support |
|--------|---------|
| `.class` | Yes |
| `.jar` | Yes |
| `.apk` | Yes |
| `.dex` | Yes |
| `.xapk` | Yes |
| `.war` | Yes |

## GUI Workflow

1. Drag-and-drop file into BCV
2. Select decompiler: View Pane → View 1/2/3
3. Browse resource tree (left panel)
4. Compare decompiler output (right panel)
5. Edit Java/bytecode if needed
6. Export: File → Export

## Plugin Commands

```javascript
// Example: Find string constants
var classes = api.getClasses();
for (var i = 0; i < classes.size(); i++) {
    api.log(classes.get(i).name);
}
```

## Batch Processing

```bash
#!/bin/bash
for jar in *.jar; do
    name=$(basename "$jar" .jar)
    java -jar BCV.jar -decompiler cfr -i "$jar" \
        -o "output/$name/" -t all -nowait
done
```

## Performance

```bash
# Increase heap for large files
java -Xmx4G -jar bytecode-viewer.jar

# Use System Theme if UI lags
# View → Visual Settings → Window Theme → System Theme
```

## Tips

- Compare multiple decompilers for best output
- Use `-nowait` for automated processing
- Increase heap with `-Xmx` for large JARs
- Use CLI for batch operations
- Check BCV Discord for support
