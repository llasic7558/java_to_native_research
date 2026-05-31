# LLM-Ported Java Benchmark Study

This repository is for Java
benchmarks compare with native ports produced with LLM assistance. The central
goal is to compare native performance vs GC managed code and to 
question that on modern workloads is there possibly an undetermined gap. This will be done 
 by comparing selected DaCapo Java benchmarks
against workload-equivalent implementations in C, C++, and maybe Rust.

The project builds on the prior thesis in
[`old_paper/final_Honors_Thesis.pdf`](old_paper/final_Honors_Thesis.pdf). But is aiming to 
shift away from a purely GC vs EMM debate as the current methodology and results alone can not 
answer that question fully. Instead aiming to examine native performance and GC performance in a way 
that could be done before. 

## Research Focus

The paper asks how close modern Java execution gets to native execution when
the native code is written to preserve the benchmark workload rather than just
the output format. The comparison is not a pure isolation of garbage-collection
cost, because the native ports do not preserve the exact JVM object layouts,
JIT behavior, or runtime services. 

Core questions:
- How do modern Java collectors compare with native C and C++ baselines?
- Does increasing Java heap space close the gap to native execution?
- How do Java and native versions differ in peak RSS, dynamic work, and
  hardware-counter behavior?
- How much does collector choice matter once Java is tuned fairly?
- How much of the observed gap comes from implementation structure rather than
  memory management alone?
- Can additional C++, and potentially Rust, ports make the comparison more
  faithful by preserving more of the original Java structure?

## Repository Layout

| Path | Purpose |
| --- | --- |
| [`benchmarks/`](benchmarks/) | Main benchmark artifact: C and C++ ports, tests, CMake files, perf harness, plots, results, and benchmark-specific notes. |
| [`old_paper/`](old_paper/) | Prior thesis PDF used as the starting point for this paper. |
| [`new_paper/`](new_paper/) | Placeholder for the revised paper draft and related writing artifacts. |
| [`How_to_Convert_benchmark.md`](How_to_Convert_benchmark.md) | A plan for how to port benchmarks for an LLM agent |
| [`TODO.md`](TODO.md) | To Do list of what to accomplish |

The benchmark implementation details live primarily in
[`benchmarks/README.md`](benchmarks/README.md). The performance harness is
documented in
[`benchmarks/perf_harness/README.md`](benchmarks/perf_harness/README.md).

## Current Benchmark Artifact

The existing artifact includes native ports for DaCapo-derived workloads such
as `luindex`, `lusearch`, `sunflow`, `xalan`, `h2`, `graphchi`, `biojava`, and
`zxing`, plus experimental C++ ports for some Lucene and BioJava workloads.
The benchmark suite includes:

- C ports under `benchmarks/c-*`
- C++ ports and probes under `benchmarks/cpp-*`

## Methodology Summary

The porting workflow described in the prior thesis has four stages:

1. Identify the DaCapo benchmark path and any large upstream Java libraries
   that actually perform the workload.
2. Write a design specification covering the Java workload, native
   architecture, expected behavior, and validation strategy.
3. Use an LLM-assisted coding workflow to implement the native port in smaller
   modules or classes.
4. Validate behavior against Java reference outputs using golden files,
   byte-identical outputs, digest comparisons, structured equivalence checks,
   deterministic replay, or per-input oracle tables.

## Running the Benchmark Artifact

Most commands should be run from `benchmarks/`.

```bash
cd benchmarks
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
ctest --test-dir build --output-on-failure
```

The full Java comparison requires a local DaCapo 23.11 MR2 Chopin jar and
extracted data directory inside `benchmarks/`:

```text
benchmarks/dacapo-23.11-MR2-chopin.jar
benchmarks/dacapo-23.11-MR2-chopin/
```

The perf harness is Linux-only and requires `perf`, `/proc`, a JDK, Python
dependencies, and permissions for unprivileged performance counters.

```bash
cd benchmarks
python3 -m perf_harness.run_harness --size small --trials 5 --dry-run
python3 -m perf_harness.run_harness --size small --trials 5 --benchmarks luindex,graphchi
python3 -m perf_harness.aggregate perf_harness/results/<timestamp>
python3 -m perf_harness.plots perf_harness/results/<timestamp>
```

## Paper Roadmap

Near-term writing and artifact tasks:

- move the revised paper draft into `new_paper/`
- turn the prior thesis into a tighter paper with a clearer claim boundary
- document the LLM porting workflow in `How_to_Convert_benchmark.md`
- distinguish strong benchmark cases from caveated and anomalous cases
- add or improve C++ ports where object structure matters
- evaluate whether Rust ports can provide a useful additional explicit-memory
  or ownership-based baseline
- make benchmark validation evidence easy to audit from the paper
