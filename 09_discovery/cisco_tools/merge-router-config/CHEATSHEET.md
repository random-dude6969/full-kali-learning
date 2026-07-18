# merge-router-config CHEATSHEET

## Quick Reference

### Merge Two Configs
```bash
merge-router-config -a config1.txt -b config2.txt -o merged.txt
```

### Compare Configs
```bash
merge-router-config -a config1.txt -b config2.txt --diff
```

### Diff Formats
```bash
--unified       # Unified diff
--side-by-side  # Side by side
--context       # Context diff
```

### Options
```bash
-a file1      # First config
-b file2      # Second config
-d dir        # Directory of configs
-o file       # Output file
--diff        # Compare mode
--ignore-comments   # Ignore comments
--ignore-whitespace # Ignore whitespace
```

### Apply Template
```bash
merge-router-config -a base.txt -t template.txt -o output.txt
```

---

## Target: 192.168.1.x

```bash
merge-router-config -a current.txt -b backup.txt --diff
```
