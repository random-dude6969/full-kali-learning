---
tool_name: afl-fuzz
display_name: AFL (American Fuzzy Lop) - Coverage-Guided Fuzzer
mitre_tactic: resource_development
mitre_tactic_id: TA0042
mitre_subcategory: fuzzing
install_command: "sudo apt install afl++"
version: "2.57b (archived); AFL++ actively maintained"
github_repo: "https://github.com/google/AFL"
kali_tools_page: "https://www.kali.org/tools/afl++/"
difficulty_level: advanced
requires_root: false
gui_available: false
tags: [fuzzing, coverage-guided, genetic-algorithm, binary-analysis, crash-detection]
related_tools: [afl++, libfuzzer, honggfuzz, spike]
---

# AFL (American Fuzzy Lop) - Coverage-Guided Fuzzer

## 1. Introduction & Overview

American Fuzzy Lop (AFL) is a security-oriented, coverage-guided fuzzer that uses a genetic algorithm to automatically discover clean, interesting test cases that trigger new internal states in the targeted binary. Created by Michał Zalewski (lcamtuf) at Google, AFL revolutionized fuzzing by introducing lightweight instrumentation and edge-coverage feedback, making it possible to find bugs that blind fuzzers would take years to discover.

**Created:** 2013 by Michał Zalewski (lcamtuf@google.com)
**License:** Apache 2.0
**Status:** Archived (March 2024). AFL++ (https://github.com/AFLplusplus/AFLplusplus) is the actively maintained successor.

### What It Does

AFL instruments a target binary at compile time (or via QEMU for binary-only targets) and tracks code coverage during execution. It maintains a queue of test cases, mutates them, and feeds mutations back into the target. When a mutation discovers a new code path (new "edge" in the coverage map), it's added to the queue for further fuzzing. This feedback loop enables AFL to efficiently explore deep program states.

### The AFL Algorithm

1. Load initial test cases into the queue
2. Select next test case from queue
3. Trim test case to minimum viable size
4. Apply mutation strategies (bit flips, byte operations, arithmetic, dictionary)
5. Execute mutated input through instrumented binary
6. Check if new code path was discovered — if so, add to queue
7. Repeat from step 2

### When to Use

- **Vulnerability discovery in C/C++ programs**: AFL excels at finding memory corruption bugs
- **File format fuzzing**: Images (PNG, JPEG), documents (PDF, DOCX), archives (ZIP, RAR)
- **Protocol fuzzing**: When paired with a network-to-stdin proxy
- **Library fuzzing**: Any library that processes untrusted input
- **Regression testing**: Feed known crashers back to prevent regressions

### When NOT to Use

- Target requires GUI interaction or complex state machines without modification
- Target uses encryption, checksums, or compression around the data you need to fuzz (workarounds exist but require code changes)
- You need to fuzz network services directly (requires architectural modification or Preeny)
- Real-time analysis needed — AFL processes statically, not dynamically

### Legal & Ethical

AFL is a defensive security tool used by researchers and developers worldwide. Fuzz your own software, open-source projects with permission, or systems under authorized security testing contracts. Fuzzing production systems without authorization may violate laws and cause service disruptions.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Coverage-guided** | Uses code coverage feedback to guide mutation strategy |
| **Edge coverage** | Tracks transitions between basic blocks, not just block coverage |
| **Instrumentation** | Compiler-level insertion of coverage tracking code |
| **Queue** | Set of test cases that discovered new paths |
| **Corpus** | Collection of interesting test cases for seeding |
| **Deterministic fuzzing** | Phase where AFL applies systematic, non-random mutations |
| **Havoc mode** | Randomized mutation phase that applies multiple strategies |
| **Splice** | Combines two test cases to create new inputs |
| **Trim** | Reduces test case size while preserving behavior |
| **Dictionary** | File of tokens/keywords to inject during mutation |

## 2. Installation & Setup

### System Requirements

- Linux (x86_64, i386, ARM, AArch64)
- GCC or Clang with instrumentation support
- 2+ CPU cores recommended (1 core per AFL instance)
- Sufficient RAM for target programs
- SSD recommended for heavy fuzzing workloads

### Installation

**From Kali repositories (AFL++ recommended):**

```bash
sudo apt install afl++
```

**Expected output:**
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  afl++
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 15.8 MB of archives.
After this operation, 62.4 MB of additional disk space will be used.
Get:1 http://kali.download/kali kali-rolling/main amd64 afl++ 4.21a-1 [15.8 MB]
Fetched 15.8 MB in 1s (12.3 MB/s)
Selecting previously unselected package afl++.
Preparing to unpack .../afl++_4.21a-1_amd64.deb ...
Unpacking afl++ (4.21a-1) ...
Setting up afl++ (4.21a-1) ...
```

**From source (original AFL — archived):**

```bash
git clone https://github.com/google/AFL.git
cd AFL
make
sudo make install
```

**Expected output:**
```
[*] Compiling afl-gcc...
afl-as.c: In function 'main':
afl-as.c:445:15: warning: implicit declaration of function 'gets'
[+] Compiling everything, sit back and relax.
[*] Testing binary on a trivial program...
[+] All set, you can start fuzzing now.
```

### Configuration

**Core settings (in `config.h` for original AFL):**

| Setting | Default | Description |
|---------|---------|-------------|
| `MAP_SIZE` | 65536 | Coverage map size |
| `MAX_FILE` | 1048576 | Maximum input file size (1 MB) |
| `MEM_LIMIT` | 50 MB | Memory limit for target process |
| `TIMEOUT` | 1000 ms | Execution timeout per test case |

**Environment variables:**

| Variable | Description |
|----------|-------------|
| `AFL_HARDEN` | Enable stack protector and other hardening |
| `AFL_ASAN` | Enable AddressSanitizer builds |
| `AFL_USE_ASAN` | Auto-instrument with ASAN |
| `AFL_SKIP_CPUFREQ` | Skip CPU frequency checks |
| `AFL_NO_UI` | Disable terminal UI (for CI/CD) |
| `AFL_FAST_CAL` | Faster calibration |

### Verification

```bash
afl-fuzz --version
```

**Expected output:**
```
afl-fuzz++4.21a (original AFL by lcamtuf)
```

## 3. Basic Usage

### Instrumenting a Target (Source Available)

```bash
CC=afl-gcc CXX=afl-g++ ./configure
make clean all
```

**Explanation:** Replaces GCC/Clang with AFL's instrumented wrappers to inject coverage tracking into the compiled binary.

**For C++ programs:**
```bash
CXX=afl-g++ CC=afl-gcc ./configure --disable-shared
```

### Creating Initial Test Cases

```bash
mkdir -p testcases/
echo "test" > testcases/test1.txt
echo "AAAA" > testcases/test2.txt
```

**Explanation:** AFL needs at least one valid input file to start fuzzing. Keep files small (under 1 KB ideal).

### Running AFL (stdin input)

```bash
afl-fuzz -i testcases/ -o findings/ ./target_binary @@
```

**Explanation:** Fuzzes `target_binary` which reads from a file specified by `@@`. AFL substitutes each test case for `@@`.

### Running AFL (file input)

```bash
afl-fuzz -i testcases/ -o findings/ ./target_binary
```

**Explanation:** For programs that read from stdin, AFL pipes mutated data directly.

### With Dictionary

```bash
afl-fuzz -i testcases/ -o findings/ -x /usr/share/afl/dictionaries/json.dict ./json_parser @@
```

**Explanation:** Seeds the fuzzer with JSON tokens to improve parsing coverage.

### With Time Limit and Memory Limit

```bash
afl-fuzz -i testcases/ -o findings/ -t 5000 -m 200 ./target_binary @@
```

**Explanation:** Sets 5-second timeout and 200 MB memory limit per execution.

### Resuming a Previous Session

```bash
afl-fuzz -i- -o findings/ ./target_binary @@
```

**Explanation:** The `-i-` flag tells AFL to resume from the existing output directory without a fresh seed corpus.

### Interpreting the Status Screen

The AFL TUI shows:
- **Cycle progress**: How many queue rounds completed
- **Total paths**: Unique execution paths discovered
- **Unique crashes**: Number of unique crashing inputs found
- **Unique hangs**: Inputs causing timeout
- **Exec speed**: Executions per second (higher is better)
- **Map coverage**: Percentage of instrumented edges hit

### Flags Reference

| Flag | Description |
|------|-------------|
| `-i dir` | Input directory with seed test cases |
| `-i-` | Resume from existing output directory |
| `-o dir` | Output directory for findings |
| `-x dict` | Fuzzer dictionary file |
| `-t ms` | Execution timeout (default: 1000ms) |
| `-m MB` | Memory limit (default: 50MB) |
| `-d` | Skip deterministic fuzzing (fast but shallow) |
| `-Q` | QEMU mode (binary-only targets) |
| `-n` | Disable instrumentation (dumb fuzzer mode) |
| `-C` | Crash exploration mode |
| `-f file` | Write mutated data to specific file |
| `-T title` | Set process title for parallel fuzzing |
| `-M name` | Main fuzzer instance (for parallel mode) |
| `-S name` | Secondary fuzzer instance |

## 4. Intermediate Usage

### Parallel Fuzzing

```bash
# Terminal 1 - Main instance
afl-fuzz -i testcases/ -o findings/ -M main ./target_binary @@

# Terminal 2 - Secondary instance
afl-fuzz -i testcases/ -o findings/ -S secondary1 ./target_binary @@

# Terminal 3 - Another secondary
afl-fuzz -i testcases/ -o findings/ -S secondary2 ./target_binary @@
```

**Explanation:** Multiple AFL instances share the same output directory, automatically synchronizing findings.

### Fuzzing Libraries

```bash
# Write a simple harness
cat > fuzz_lib.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include "target_library.h"

int main(int argc, char **argv) {
    FILE *f = fopen(argv[1], "rb");
    if (!f) return 1;
    fseek(f, 0, SEEK_END);
    long size = ftell(f);
    rewind(f);
    char *buf = malloc(size);
    fread(buf, 1, size, f);
    fclose(f);
    target_library_function(buf, size);
    free(buf);
    return 0;
}
EOF

# Compile with AFL
afl-gcc -o fuzz_lib fuzz_lib.c -L. -ltarget -static

# Create seed
echo "test" > seed.txt

# Fuzz
afl-fuzz -i seeds/ -o findings/ ./fuzz_lib @@
```

**Explanation:** Wrap library functions in a harness that reads input from a file, then fuzz the harness.

### Using AFL-Tmin for Minimization

```bash
afl-tmin -i crash_input -o minimized_input -- ./target_binary @@
```

**Explanation:** Reduces a crashing test case to its minimal form while preserving the crash.

### Using AFL-Showmap for Coverage Analysis

```bash
afl-showmap -t 1000 ./target_binary -- input_file
```

**Explanation:** Displays the coverage map for a specific input, useful for understanding what code paths are exercised.

### Using AFL-Cmin for Corpus Minimization

```bash
afl-cmin -i corpus/ -o minimized_corpus/ -- ./target_binary @@
```

**Explanation:** Reduces a large seed corpus to the smallest set that maintains the same coverage.

### Binary-Only Fuzzing with QEMU

```bash
# Build QEMU support
cd qemu_mode
./build_qemu_support.sh

# Fuzz with QEMU instrumentation
afl-fuzz -i testcases/ -o findings/ -Q ./binary_only_target @@
```

**Explanation:** QEMU mode instruments binary-only targets at runtime, approximately 2-5x slower than compile-time instrumentation.

### Dictionaries

```bash
afl-fuzz -i testcases/ -o findings/ -x /usr/share/afl/dictionaries/http.dict ./http_parser @@
```

**Explanation:** Injects HTTP-related tokens (GET, POST, Content-Length, etc.) during mutation, improving protocol parser coverage.

## 5. Advanced Usage

### ASAN Builds for Deeper Bug Detection

```bash
AFL_USE_ASAN=1 afl-gcc -o target target.c
afl-fuzz -i testcases/ -o findings/ -m 2048 ./target @@
```

**Explanation:** AddressSanitizer detects memory corruption that might not immediately crash. Use `-m 2048` (or higher) because ASAN increases memory usage.

### Crash Exploration Mode

```bash
afl-fuzz -i crash_input/ -o crash_exploration/ -C ./target_binary @@
```

**Explanation:** Takes a crashing test case and explores all paths that maintain the crash state. Useful for understanding attacker control over the faulting address.

### Custom Mutation Strategies (AFL++)

```bash
afl-fuzz -i testcases/ -o findings/ -c ./target_binary @@ -p fast
```

**Explanation:** AFL++ supports power schedules (fast, coe, explore, rare) that adjust mutation aggressiveness based on path discovery rate.

### Automating Bug Triage

```bash
#!/bin/bash
find findings/crashes/ -type f | while read crash_file; do
    echo "=== Testing: $crash_file ==="
    gdb -batch -ex run -ex bt ./target_binary < "$crash_file"
    echo ""
done
```

**Explanation:** Automates crash triage by running each crashing input through GDB and capturing backtraces.

### Monitoring with afl-plot

```bash
afl-plot findings/ plot_output/
```

**Explanation:** Generates gnuplot-based graphs showing coverage over time.

### Using libdislocator

```bash
AFL_USE_DISLOCATOR=1 afl-gcc -o target target.c
afl-fuzz -i testcases/ -o findings/ ./target @@
```

**Explanation:** libdislocator replaces the allocator to detect heap corruption with high probability.

## 6. Real-World Scenarios

### Scenario 1: File Format Fuzzing — PDF Parser

**Target:** Poppler PDF library

```bash
# Create minimal PDF seed
echo "%PDF-1.0
1 0 obj
<< /Type /Catalog >>
endobj" > seed.pdf

# Instrument poppler
CC=afl-gcc CXX=afl-g++ ./configure --disable-shared
make clean all

# Create harness (pdftotext reads files)
mkdir -p testcases/ findings/
cp seed.pdf testcases/

# Run AFL
afl-fuzz -i testcases/ -o findings/ ./pdftotext @@
```

**Expected result:** Hundreds of crashes within hours, revealing memory corruption in PDF parsing.

### Scenario 2: Network Protocol Fuzzing

**Target:** Custom TCP server on port 8080

```bash
# Use socat to bridge stdin to network
cat > /tmp/fuzz_tcp.sh << 'EOF'
#!/bin/bash
socat TCP-LISTEN:8080,reuseaddr,fork SYSTEM:"cat | ./target_binary"
EOF
chmod +x /tmp/fuzz_tcp.sh

# Create TCP proxy harness
cat > tcp_harness.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    char buf[65536];
    int n = read(STDIN_FILENO, buf, sizeof(buf));
    if (n > 0) {
        // Call target function with buf
        target_protocol_handler(buf, n);
    }
    return 0;
}
EOF

afl-gcc -o tcp_harness tcp_harness.c
afl-fuzz -i seeds/ -o findings/ ./tcp_harness
```

**Explanation:** Network services need architectural modification to accept stdin input for AFL fuzzing.

### Scenario 3: Image Processing Library

**Target:** libpng

```bash
CC=afl-gcc CXX=afl-g++ ./configure --disable-shared
make clean all

# Seeds: minimal valid PNGs
mkdir -p testcases/
convert -size 10x10 xc:red testcases/seed.png  # ImageMagick

afl-fuzz -i testcases/ -o findings/ -x /usr/share/afl/dictionaries/png.dict ./pngtest @@
```

### Scenario 4: Continuous Integration

```bash
# In CI pipeline
export AFL_NO_UI=1
export AFL_SKIP_CPUFREQ=1
timeout 3600 afl-fuzz -i testcases/ -o findings/ ./target_binary @@

# Check results
if [ -d findings/crashes ]; then
    echo "CRASHES FOUND:"
    ls findings/crashes/
    exit 1
fi
```

**Explanation:** Run AFL in CI with `AFL_NO_UI` to disable the interactive interface and use timeout to bound execution.

## 7. Detection & Defense

### How Defenders Detect

- **CPU usage**: AFL pegs a single core at 100% — anomalous sustained CPU load
- **File system activity**: Thousands of short-lived files in the output directory
- **Network monitoring**: Unusual connection patterns if fuzzing network services
- **System calls**: High `execve()` count for the target binary
- **Memory pressure**: Target may consume excessive RAM under fuzzing

### Mitigation

- Deploy AddressSanitizer (ASAN) to make crashes harder to exploit
- Use code hardening: stack protectors, PIE, RELRO
- Monitor for `afl-fuzz` process signatures in host-based IDS
- Implement resource limits (cgroups, ulimit) to prevent denial of service

### Blue Team Response

```bash
# Monitor for AFL-related processes
ps aux | grep -E "afl-|afl_" 
find /tmp /var/tmp -name "*.fuzz*" -o -name "fuzz_*" 2>/dev/null

# Check for instrumentation artifacts
find /usr/local/bin /opt -name "afl-*" 2>/dev/null
```

## 8. Troubleshooting

### Common Errors

**"Whoops, your system is configured to send core dumps to AFL's output directory"**

```bash
echo core > /proc/sys/kernel/core_pattern
```

**Explanation:** AFL requires core dumps not go to the output directory. This command redirects them to `/core`.

**"CPU frequency scaling appears to be causing problems"**

```bash
export AFL_SKIP_CPUFREQ=1
```

**Explanation:** On systems without frequency scaling support, AFL may refuse to start.

**"No instrumentation detected"**

Ensure the binary was compiled with `afl-gcc`/`afl-g++` or instrumented with `afl-clang-fast`.

**Low execution speed:**

```bash
afl-gotcpu  # Check available CPU
# Ensure no other CPU-intensive tasks are running
# Use -m to increase memory limit if ASAN is enabled
```

### Performance Optimization

- **SSD storage**: Dramatically improves I/O-heavy fuzzing
- **Disable hyper-threading**: Prevents contention for shared cache
- **Pin AFL to specific CPU cores**: `taskset -c 0 afl-fuzz ...`
- **Use `-d` flag**: Skip deterministic phase for faster initial coverage
- **Parallel fuzzing**: Use `-M`/`-S` flags across multiple cores

### FAQ

**Q: AFL vs AFL++?**
A: AFL (original) is archived. AFL++ is the community-maintained successor with hundreds of improvements. Always use AFL++ on new projects.

**Q: Can I fuzz interpreted languages?**
A: Yes, but instrument the interpreter, not the script. Fuzz Python by instrumenting the CPython binary with Python script inputs.

**Q: How long should I fuzz?**
A: Minimum 24 hours for meaningful results. Many real-world bugs are found after 1+ weeks of continuous fuzzing.

**Q: What about network services?**
A: Requires modification to read from stdin, or use Preeny (https://github.com/zardus/preeny) to convert network input to stdin.

---

## Resources

- **Original AFL**: https://github.com/google/AFL (archived)
- **AFL++ (recommended)**: https://github.com/AFLplusplus/AFLplusplus
- **AFL Documentation**: `docs/` subdirectory in the AFL repository
- **Fuzzing Book**: https://www.fuzzingbook.org/
- **AFL Training**: https://github.com/roots-lab/AFLTraining
- **Fastly Blog — How to Fuzz a Server**: https://www.fastly.com/blog/how-to-fuzz-server-american-fuzzy-lop
- **Preeny**: https://github.com/zardus/preeny (network-to-stdin adapters)
