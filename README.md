# Standard Clojure Style (jank)

A native Clojure code formatter running on [jank](https://jank-lang.org/), a Clojure dialect hosted on C++ and LLVM.

Ported from [standard-clojure-style-js](https://github.com/oakmac/standard-clojure-style-js).

## Status

All test suites passing with 100% test coverage:
* **Parser suite:** 141 / 141 tests passing
* **Parse-NS suite:** 83 / 83 tests passing
* **Format suite:** 235 / 235 tests passing

Total: **459 / 459 tests passing**.

---

## Features

* **Zero External Binary Dependencies:** Implemented in pure Clojure running natively on `jank`.
* **Complete Format Specification:** Produces identical formatting to upstream Standard Clojure Style.
* **Deterministic Namespace Pretty-Printing:** Complete sorting and structure alignment for `:refer-clojure`, `:require-macros`, `:require`, `:import`, and `:gen-class`, with full reader conditional (`#?(...)`, `#?@(...)`) support.
* **Command-Line Interface:** `bin/standard-clj` provides `fix`, `check`, `list`, and stdin (`-`) pipelines.

---

## Installation & Requirements

* Requires `jank` (v0.1+) installed and available on `PATH`.

Make the CLI executable:

```bash
chmod +x bin/standard-clj
```

Optionally symlink `bin/standard-clj` to `/usr/local/bin` or a directory on your `$PATH`.

---

## Command Line Usage

### Format files in place (`fix`)

```bash
bin/standard-clj fix src/ my_file.clj
```

### Check if files are formatted (`check`)

Exits with `0` if all files match Standard Clojure Style, or `1` if any file requires formatting:

```bash
bin/standard-clj check src/
```

### Stdin formatting (`fix -`)

Pipe Clojure code to `standard-clj` to format it directly to stdout:

```bash
echo '(ns foo (:require [b] [a]))' | bin/standard-clj fix -
```

Output:
```clojure
(ns foo
  (:require
   [a]
   [b]))
```

### List files that would be processed (`list`)

```bash
bin/standard-clj list src/
```

---

## Library API

Add the `src` directory to your `--module-path`:

```clojure
(ns my-app.core
  (:require [standard-clojure-style.core :as scs]))

;; Format code string
(scs/format "(let [x 1 y 2] (+ x y))")
;; => {:status "success" :out "(let [x 1\n      y 2]\n  (+ x y))"}

;; Parse code to AST
(scs/parse "(defn add [a b] (+ a b))")

;; Parse namespace form
(scs/parse-ns "(ns com.example (:require [foo.bar :as f]))")
```

---

## Running Tests

Run each test suite via `jank`:

```bash
# Parser tests (141 tests)
jank --module-path src:test:test_cases run test/standard_clojure_style/parser_test.jank

# Namespace parser tests (83 tests)
jank --module-path src:test:test_cases run test/standard_clojure_style/parse_ns_test.jank

# Formatter tests (235 tests)
jank --module-path src:test:test_cases run test/standard_clojure_style/format_test.jank
```

---

## Architecture & jank Runtime Considerations

During the port to `jank` (v0.1 / LLVM), several runtime nuances were handled:

1. **Pure String Functions (`standard-clojure-style.util`):**
   `clojure.string` in the default jank runtime relies on `native/raw`, which is not exposed during user code evaluation. A self-contained, pure-Clojure string utility suite was created with correct string slicing and index mechanics.
2. **Associative Vector Updates:**
   `persistent_vector` in jank v0.1 does not support associative `assoc`. Vector updates are performed using pure slicing/reconstruction helpers (`assoc*`, `assoc-vec`, `assoc-in`, `update*`).
3. **Empty Lazy Sequences & Reduce:**
   Jank's `reduce` iterates over lazy sequences before validating whether the sequence is empty, which caused empty lazy sequences (e.g. `(partition 2 [])`, `(filter ...)`) to reduce into `[nil]`. Pure eager vector operations (`u/filterv*`, `u/mapv*`, `u/removev*`, `u/partition-pairs`) ensure correct behavior.
4. **`swap!` Arity Protection:**
   Jank's `swap!` supports up to 2 extra parameters before delegating to variable arity via `apply`. Multi-key updates are isolated in single-argument updater functions.
5. **Exception Handling:**
   `try` expressions must be the root form of functions in jank to avoid compiler/LLVM optimization crashes.

---

## License

ISC License (matching upstream [standard-clojure-style-js](https://github.com/oakmac/standard-clojure-style-js)).
