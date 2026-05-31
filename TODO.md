# TODO: Paper Plan

## 1. Draft Abstract and Methodology

## 2. Finish C++ Ports/Clean Up Any Previous Benchmarks 

Fix Existing C++ ports:

- `cpp-luindex`
- `cpp-lusearch`
- `cpp-sunflow`
- `cpp-biojava`

Port these C benchmarks to C++:

- `c-h2` -> `cpp-h2`
- `c-xalan` -> `cpp-xalan`
- `c-graphchi` -> `cpp-graphchi`
- `c-zxing` -> `cpp-zxing`

## 3. Add 4-6 More DaCapo Benchmarks

Maybe:

- `batik`: SVG/rendering, object-heavy document and graphics state.
- `tradebeans`: transactional business objects.
- `tradesoap`: SOAP/XML serialization and service-stack churn.
- `tomcat`: request parsing, framework object graphs, request lifetimes.
- `jython`: interpreter object model and dispatch.
- `eclipse`: large application-style codebase, high porting cost.

For each benchmark do the same:
- A C port focused on explicit memory management and minimal abstraction overhead.
- A C++ port that is a Java-like variant

## 4. Add Architecture Fidelity Experiment

Use `biojava` and do have different levels like so: 
- Level 0: C baseline with flat structs/functions, manual ownership, minimal dispatch, and buffer reuse.
- Level 1: idiomatic C++ value model with classes, RAII, `std::vector`, and `std::string`.
- Level 2: direct class translation with one C++ class per Java class where practical.
- Level 3: Java-like heap-object C++ with pointer/smart-pointer fields, factory allocation, object identity, and virtual dispatch.
- Level 4: runtime-structure stress variant with Java-like temporary allocations, boxed/wrapper objects where applicable, and optional arena/tracing modes.

## 5. Design Java -> C# -> Unity IL2CPP Experiment? 

Goal: test a production managed-to-native lowering path as another way to produce "Java-like C++."

Pipeline:

1. Start from a Java benchmark kernel.
2. Use a code agent to translate Java to architecture-preserving C#.
3. Validate under plain .NET.
4. Move the C# into a Unity-compatible harness.
5. Build through Unity IL2CPP to generated C++.
6. Compile/run the native binary.
7. Compare Java, .NET C#, IL2CPP C++, and hand-written C++.

## 6. Collect Results 

## 7. Write Rough Draft

## 8. Get Feedback