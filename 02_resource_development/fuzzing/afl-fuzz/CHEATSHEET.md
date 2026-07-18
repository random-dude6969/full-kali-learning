# AFL (American Fuzzy Lop) Cheatsheet

## Quick Start

```bash
# Instrument and compile
CC=afl-gcc ./configure && make clean all

# Create seed
echo "test" > seed.txt

# Run AFL
afl-fuzz -i seeds/ -o findings/ ./target @@

# Resume
afl-fuzz -i- -o findings/ ./target @@
```

## Compilation

```bash
CC=afl-gcc ./configure           # C programs
CXX=afl-g++ ./configure          # C++ programs
AFL_USE_ASAN=1 afl-gcc ...       # With AddressSanitizer
AFL_HARDEN=1 make                # Enable hardening
```

## AFL Commands

| Command | Description |
|---------|-------------|
| `afl-fuzz` | Main fuzzer |
| `afl-gcc` | GCC wrapper |
| `afl-g++` | G++ wrapper |
| `afl-tmin` | Minimize test cases |
| `afl-showmap` | Show coverage map |
| `afl-cmin` | Minimize corpus |
| `afl-analyze` | Analyze input |
| `afl-gotcpu` | Check CPU availability |
| `afl-whatsup` | Report status |
| `afl-plot` | Generate graphs |

## Key Flags

| Flag | Description |
|------|-------------|
| `-i dir` | Input seed directory |
| `-i-` | Resume from existing output |
| `-o dir` | Output directory |
| `-x dict` | Dictionary file |
| `-t ms` | Timeout (default: 1000ms) |
| `-m MB` | Memory limit (default: 50MB) |
| `-d` | Skip deterministic phase |
| `-Q` | QEMU mode (binary-only) |
| `-n` | Dumb fuzzer (no instrumentation) |
| `-C` | Crash exploration mode |
| `-f file` | Write to specific file |
| `-M name` | Main instance |
| `-S name` | Secondary instance |

## Parallel Fuzzing

```bash
# Main instance
afl-fuzz -i seeds/ -o findings/ -M main ./target @@

# Secondary instances
afl-fuzz -i seeds/ -o findings/ -S sec1 ./target @@
afl-fuzz -i seeds/ -o findings/ -S sec2 ./target @@
```

## Environment Variables

```bash
AFL_HARDEN=1                # Enable hardening
AFL_USE_ASAN=1              # AddressSanitizer
AFL_SKIP_CPUFREQ=1          # Skip CPU checks
AFL_NO_UI=1                 # Disable UI (CI)
AFL_FAST_CAL=1              # Fast calibration
AFL_PERSISTENT=1            # Persistent mode
```

## Corpus Management

```bash
afl-cmin -i corpus/ -o min_corpus/ -- ./target @@   # Minimize corpus
afl-tmin -i crash_input -o min_crash -- ./target @@ # Minimize crash
afl-showmap ./target -- input_file                   # Show coverage
```

## Useful Patterns

```bash
# Dictionary fuzzing
afl-fuzz -i seeds/ -o findings/ -x dicts/json.dict ./parser @@

# Network service (via socat)
socat TCP-LISTEN:8080,reuseaddr,fork SYSTEM:"cat | ./target"

# Library fuzzing (write harness)
cat > fuzz_lib.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char **argv) {
    FILE *f = fopen(argv[1], "rb");
    fseek(f, 0, SEEK_END);
    long sz = ftell(f); rewind(f);
    char *buf = malloc(sz);
    fread(buf, 1, sz, f); fclose(f);
    target_function(buf, sz);
    free(buf); return 0;
}
EOF
afl-gcc -o fuzz_lib fuzz_lib.c -static
afl-fuzz -i seeds/ -o findings/ ./fuzz_lib @@
```

## Crash Analysis

```bash
# Quick triage
ls findings/crashes/
file findings/crashes/*

# Backtrace each crash
for f in findings/crashes/id:*; do
    echo "=== $f ==="
    gdb -batch -ex "run < $f" -ex "bt" ./target
done
```

## CI/CD Integration

```bash
export AFL_NO_UI=1
export AFL_SKIP_CPUFREQ=1
timeout 3600 afl-fuzz -i seeds/ -o findings/ ./target @@
[ -d findings/crashes ] && echo "CRASHES FOUND" && exit 1
```

## Tips

- Seed files should be < 1 KB
- Use `-d` for quick initial coverage
- Run on SSD for best I/O
- One AFL instance per CPU core
- Minimum 24h for meaningful results
- Use AFL++ for new projects (actively maintained)
