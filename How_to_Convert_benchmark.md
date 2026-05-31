# DaCapo Benchmark Conversion Plan

Use this as a compact planning prompt for converting one DaCapo Java benchmark
to native C/C++ ports. Keep the first target single-threaded and validated on
DaCapo `small` and `default`.

## Agent Task

For the chosen DaCapo benchmark:

1. Analyze the Java benchmark and upstream libraries.
2. If no C port exists, write a C port plan.
3. Write a C++ port plan.
4. Define validation tests, run them, fix mismatches, and record evidence.

Do not trust timing results until validation passes or the remaining mismatch
is explicitly documented.

## 1. Java Benchmark Analysis

Read the DaCapo driver, input config, size presets, data files, and upstream
library source/jars. Identify the real workload, not just the wrapper.

Record:

- DaCapo version and benchmark name
- Java entry point
- upstream libraries and versions
- source files/jars inspected
- `small` and `default` inputs
- Java command lines for reference runs
- benchmark stages, such as parse, index, search, transform, render, or solve
- visible outputs: stdout digest, files, reports, rankings, images, DB state
- hidden workload shape: allocation churn, object structure, tokenization,
  codec format, RNG seeds, sort order, floating-point/formatting rules
- Java threading behavior, even if the native first target is single-threaded

Answer in one paragraph: what is this benchmark testing?

Write these findings to `benchmarks/<port-dir>/JAVA_ANALYSIS.md` in the
created port directory, for example `benchmarks/c-<name>/JAVA_ANALYSIS.md` or
`benchmarks/cpp-<name>/JAVA_ANALYSIS.md`, so later agents can reference the
analysis without re-reading all Java sources.

## 2. C Port Plan

If `benchmarks/c-<name>/` does not exist, plan the C port first. It is the
explicit-memory baseline, so keep ownership and allocation behavior auditable.

Plan:

- scope: single-threaded `small` and `default`
- CLI: usually `-s small|default`, `-n iterations`, `--alloc-stats`, optional
  input/output overrides, and `-t 1` if a thread flag exists
- input mapping from DaCapo files to native files
- output mapping from Java-visible results to native results
- modules and data structures
- ownership/freeing rules
- allocation tracking approach
- deterministic behavior requirements
- known simplifications and benchmark-fidelity risks
- build/test commands

Milestones:

1. Smoke-test driver.
2. Parser/loader unit tests.
3. Core algorithm unit tests.
4. Java reference or golden generator.
5. End-to-end `small` validation.
6. End-to-end `default` validation.
7. Allocation/workload counters.
8. Harness integration.

## 3. C++ Port Plan

Plan C++ after the Java analysis and C scope are clear. Use C++ to preserve
Java-like object structure when it improves fidelity.

Plan:

- scope: single-threaded `small` and `default`
- C++ standard and build target
- reused repo infrastructure, such as `cpp-common`, Lucene common code, or
  existing Java reference tools
- Java class/package to C++ file/class mapping
- ownership model: value, `unique_ptr`, `shared_ptr`, arena, or explicit
  lifetime
- unsupported Java paths and proof DaCapo `small/default` do not reach them
- validation gates shared with C plus C++-specific tests
- build/test commands

For library-heavy benchmarks, port only the reachable hot set needed by DaCapo.
Cold methods may be stubs only if validation proves they are unreachable.

## 4. Validation Plan

Design validation before performance work.

Include applicable checks:

- input parsing: counts, checksums, record boundaries, malformed input
- output equivalence: byte-identical files, normalized text, TSV/JSON rows,
  image pixels, stdout digests, ranking overlap
- algorithm equivalence: tokens, terms, scores, transforms, transaction mix,
  graph updates, matrix solves, decoded text
- workload shape: documents, queries, records, transactions, rays, graph
  edges, nodes, templates, allocations by bucket
- determinism: repeated native runs match
- Java reference metadata: input hash, Java version, DaCapo version, command
- `small` and `default` gates
- negative tests for invalid/truncated/unsupported inputs

Save inspectable diffs and native/Java outputs under `test_outputs/` when
practical. Record final validation status in `FIDELITY.md`, `VALIDATION.md`,
or a phase exit note.

## 5. Correction Loop

For each failing gate:

1. Run the smallest failing test.
2. Inspect the native vs Java/golden diff.
3. Fix behavior or narrow the documented scope.
4. Re-run the focused test.
5. Re-run full `small` and `default` validation.
6. Record the result and remaining caveats.

## Deliverables

- Java workload analysis note
- `JAVA_ANALYSIS.md` in the created port directory
- C plan if no C port exists
- C++ plan
- native source/build files
- Java reference helper or golden generator when possible
- parser/algorithm unit tests
- `small` and `default` equivalence tests
- workload-shape and allocation counters where relevant
- validation/fidelity note with exact reproduction commands

## Fill-In Template

```markdown
# <benchmark> Native Port Plan

## Java Analysis
- DaCapo version:
- Java entry point:
- Upstream libraries:
- Sources inspected:
- small input:
- default input:
- Java reference command:
- Outputs:
- Workload tested:
- Threading:
- Fidelity risks:

## C Plan
- Existing C port? yes/no
- Scope:
- CLI:
- Inputs:
- Outputs:
- Modules/data structures:
- Ownership/allocation tracking:
- Simplifications:
- Validation gates:
- Build/test commands:

## C++ Plan
- Scope:
- Java classes mirrored:
- C++ files/classes:
- Reused infrastructure:
- Ownership model:
- Unsupported paths:
- Validation gates:
- Build/test commands:

## Validation
- Unit tests:
- Java/golden reference:
- small gate:
- default gate:
- Workload-shape checks:
- Determinism checks:
- Negative tests:
- Diff outputs:
- Acceptance threshold:

## Correction Notes
- Failing gate:
- Mismatch:
- Fix:
- Re-run command:
- Final result:
```
