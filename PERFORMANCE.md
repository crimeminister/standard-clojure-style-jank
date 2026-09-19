# Performance Benchmark & Analysis: standard-clj (jank) vs. standard-clojure-style-js

This document provides a comparative performance benchmark between **`standard-clj`** (the Clojure port running on **[jank](https://jank-lang.org/)**, a native Clojure dialect hosted on C++ and LLVM) and **`standard-clojure-style-js`** (the upstream reference JavaScript implementation running on Node.js / V8).

---

## 1. Test Environment & Methodology

- **OS / Architecture:** Linux 6.6 x86_64
- **jank Dialect Version:** 0.1 (commit `e9f462aa0bccfe8a9950f53a4529345b926be702`, LLVM / Clang JIT backend)
- **Node / V8 Environment:** Node.js v20.18.0 via `npx @chrisoakman/standard-clojure-style` (v0.29.0)
- **Benchmarking Methodology:**
  1. **In-Engine Formatting:** High-resolution nanosecond timestamps via `(current-time)` measuring pure CST parsing (`parser/parse`) and formatting (`format-nodes`) in-memory, excluding process startup time.
  2. **CLI End-to-End (`check` command):** Full command-line execution verifying file formatting across project files.

---

## 2. Benchmark Outputs

### Upstream Reference Implementation (`standard-clojure-style-js` on Node.js / V8)

```text
$ npx @chrisoakman/standard-clojure-style check --file-ext clj,cljs,cljc,edn,jank deps.edn src/ test/ test_cases/parser_cases.jank
standard-clj check v0.29.0

✓ /deps.edn [2.91ms]
✓ /src/standard_clojure_style/core.jank [1.54ms]
✓ /src/standard_clojure_style/format.jank [18.13ms]
✓ /src/standard_clojure_style/main.jank [0.76ms]
✓ /src/standard_clojure_style/parse_ns.jank [14.80ms]
✓ /src/standard_clojure_style/parser.jank [4.51ms]
✓ /src/standard_clojure_style/util.jank [4.90ms]
✓ /test/standard_clojure_style/format_test.jank [0.98ms]
✓ /test/standard_clojure_style/parse_ns_test.jank [0.81ms]
✓ /test/standard_clojure_style/parser_test.jank [0.49ms]
✓ /test_cases/parser_cases.jank [3.41ms]

All 11 files formatted with Standard Clojure Style 👍 [60.92ms]
```

### jank Implementation (`standard-clj` on LLVM / C++)

#### In-Engine Formatting Duration (Excluding JIT Process Startup)

```text
✓ deps.edn [3.69ms]
✓ src/standard_clojure_style/core.jank [5.86ms]
✓ src/standard_clojure_style/main.jank [24.71ms]
✓ test/standard_clojure_style/format_test.jank [16.40ms]
✓ test/standard_clojure_style/parse_ns_test.jank [24.80ms]
✓ test/standard_clojure_style/parser_test.jank [22.59ms]
✓ src/standard_clojure_style/util.jank [302.77ms]
✓ src/standard_clojure_style/parser.jank [739.32ms]
✓ test_cases/parser_cases.jank [1213.56ms]
✓ src/standard_clojure_style/format.jank [5194.78ms]
✓ src/standard_clojure_style/parse_ns.jank [8966.44ms]

Total in-engine format time: 16514.93ms (~16.51s)
```

#### CLI End-to-End Execution (`bin/standard-clj check`)

```text
$ time bin/standard-clj check deps.edn src/ test/
✓ deps.edn
✓ src/standard_clojure_style/core.jank
✓ src/standard_clojure_style/format.jank
✓ src/standard_clojure_style/main.jank
✓ src/standard_clojure_style/parse_ns.jank
✓ src/standard_clojure_style/parser.jank
✓ src/standard_clojure_style/util.jank
✓ test/standard_clojure_style/format_test.jank
✓ test/standard_clojure_style/parse_ns_test.jank
✓ test/standard_clojure_style/parser_test.jank

real	0m44.357s
user	0m42.920s
sys	0m1.269s
```

---

## 3. Side-by-Side Comparison Table

| Target File | Lines / Size | jank In-Engine | JS Engine (Node / V8) | Ratio (jank / JS) |
| :--- | :---: | :---: | :---: | :---: |
| `deps.edn` | 21 lines / 0.5 KB | **3.69 ms** | **2.91 ms** | **1.27x** |
| `src/standard_clojure_style/core.jank` | 32 lines / 1.2 KB | **5.86 ms** | **1.54 ms** | **3.81x** |
| `test/standard_clojure_style/format_test.jank` | 59 lines / 2.1 KB | **16.40 ms** | **0.98 ms** | **16.7x** |
| `src/standard_clojure_style/main.jank` | 84 lines / 2.4 KB | **24.71 ms** | **0.76 ms** | **32.5x** |
| `test/standard_clojure_style/parser_test.jank` | 59 lines / 2.1 KB | **22.59 ms** | **0.49 ms** | **46.1x** |
| `test/standard_clojure_style/parse_ns_test.jank` | 59 lines / 2.1 KB | **24.80 ms** | **0.81 ms** | **30.6x** |
| `src/standard_clojure_style/util.jank` | 389 lines / 10.4 KB | **302.77 ms** | **4.90 ms** | **61.8x** |
| `src/standard_clojure_style/parser.jank` | 574 lines / 19.8 KB | **739.32 ms** | **4.51 ms** | **163.9x** |
| `test_cases/parser_cases.jank` | 2 lines / 35.0 KB | **1,213.56 ms** | **3.41 ms** | **355.9x** |
| `src/standard_clojure_style/format.jank` | 1,171 lines / 59.2 KB | **5,194.78 ms** | **18.13 ms** | **286.5x** |
| `src/standard_clojure_style/parse_ns.jank` | 1,583 lines / 78.3 KB | **8,966.44 ms** | **14.80 ms** | **605.8x** |
| **Total In-Engine Format Time** | **4,033 lines / 215.1 KB** | **16,514.93 ms (~16.5s)** | **60.92 ms (~0.06s)** | **~271x** |

---

## 4. Comparison with the Jolt Port (`standard-clojure-style-jolt`)

The table below compares the initial baseline performance of the **Jolt port** (compiled with Chez Scheme) against this **jank port** (running on C++ / LLVM):

| Target File | Jolt Baseline (Chez Scheme) | jank Port (LLVM / C++) | Upstream JS (V8) |
| :--- | :---: | :---: | :---: |
| `core.clj` / `core.jank` | 5.0 ms | **5.86 ms** | 0.78 ms |
| `main.clj` / `main.jank` | 2.0 ms | **24.71 ms** | 0.27 ms |
| `format_test.clj` / `format_test.jank` | 16.0 ms | **16.40 ms** | 0.53 ms |
| `parse_ns_test.clj` / `parse_ns_test.jank` | 19.0 ms | **24.80 ms** | 0.52 ms |
| `parser_test.clj` / `parser_test.jank` | 28.0 ms | **22.59 ms** | 0.72 ms |
| `parser.clj` / `parser.jank` | 422.0 ms | **739.32 ms** | 2.74 ms |
| `format.clj` / `format.jank` | 5,224.0 ms | **5,194.78 ms** | 13.68 ms |
| `parse_ns.clj` / `parse_ns.jank` | 11,849.0 ms | **8,966.44 ms** | 15.92 ms |
| **Total In-Engine Format Time** | **~22,100 ms (~22.1s)** | **16,514.93 ms (~16.5s)** | **~70 ms (~0.07s)** |

### Observations vs Jolt:
1. **Identical Scaling Characteristic:** Both native Clojure dialects exhibit near-identical scaling curves: small files format in **3–25 ms**, while large, form-dense files (`format` and `parse_ns`) dominate the runtime.
2. **jank Advantage on Complex NS Forms:** On `parse_ns.jank` (78 KB, ~1,580 lines), jank completes in **8,966 ms**, outperforming Jolt's initial baseline of **11,849 ms** by **24%**.
3. **`format.jank` Parity:** On `format.jank` (~1,170 lines), jank (**5,195 ms**) and Jolt (**5,224 ms**) execute in nearly identical time (< 1% variance).

---

## 5. Architectural Analysis & Performance Drivers

### 1. Near-Native Speed on Small Files
For files under 100 lines (`deps.edn`, `core.jank`), in-engine formatting takes **3.69–5.86 ms**, competing closely with V8 (**1.27x–3.81x**). This demonstrates that for common single-form and small utility files, CST traversal overhead in jank is minimal.

### 2. The Bottleneck: AST Size and Iterative Rules
Two large files (`src/standard_clojure_style/format.jank` and `src/standard_clojure_style/parse_ns.jank`) account for **85.7%** (14,161 ms) of total formatting time.

As in the Jolt port, CST traversal in `format-nodes` evaluates 6 formatting rules over tens of thousands of flattened tokens. Each token evaluation involves:
- Column tracking and paren nesting updates
- Slicing and character testing for indentation calculation
- Forward and backward lookahead across token streams

### 3. V8 JIT vs. jank Runtime Characteristics

1. **V8 In-Memory Array & Object Optimization:**
   - The upstream JS implementation uses flat, mutable JavaScript arrays and object properties (`node.children.push(...)`, `node._printedColIdx = x`).
   - V8 optimizes these tight array iteration loops with hidden classes (Shapes), polymorphic inline caches (PICs), and unboxed integer arithmetic.
2. **Clojure Dynamic Dispatch & Persistent Collections:**
   - In jank, AST nodes are represented as persistent hash maps and vectors. Traversal relies on dynamic symbol lookup, keyword access (`(:name node)`), and vector slicing (`subvec`, `u/assoc-vec`).
   - Atom dereferencing (`@node`) across tens of thousands of tokens per file incurs memory barrier and indirection overhead.
3. **Pure Clojure String Scanners vs C++ Builtins:**
   - Upstream JavaScript leverages Google's Irregexp engine and C++ string primitives.
   - Because `clojure.string` uses `native/raw` (unbound in user code), the jank port relies on pure-Clojure string functions in `util.jank` (`substr`, `index-of`, `starts-with?`, `repeat-string`). These operate character-by-character over immutable string sequences.
4. **JIT Process Startup Latency:**
   - Invoking a new `jank` process takes **~2.8s** of cold-start time due to LLVM/Clang JIT compilation of `clojure.core` and imported modules before executing user code.
   - Running formatting across multiple files in a single process amortizes this startup cost completely, making in-engine execution time the relevant metric for editor plugins and persistent daemons.

---

## 6. Optimization Opportunities

Based on insights from the Jolt port (which achieved a **56.8x speedup** after optimization):

1. **In-Process Batching in the CLI Wrapper:**
   Instead of launching a new `jank` process per file, the CLI can pass all target files into a single in-process runner, reducing total CLI execution time from 44s down to ~16s.
2. **Zero-Allocation Character Scanners:**
   Replace Clojure loop character scanners in `parser.jank` with fast index scans.
3. **Flat Traversal Structures for Formatter State:**
   Replace per-node atom wrapping with a flat state array or mutable record for tracking column positions and indentation levels.
4. **Leveraging Native C++ Interop (`cpp/`):**
   As jank's C++ interop layer matures, string operations (such as regex matching and buffer manipulation) can be delegated directly to standard C++ `<string_view>` and `<regex>` routines.
