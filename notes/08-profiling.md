# Lecture 8: Profiling and Profile-Guided Optimization (PGO)

This lecture focuses on collecting runtime program behavior (profiling) and using that data to guide aggressive compiler optimizations.

## 1. Types of Profiling

- **Edge Profiling**: Counts how many times each edge in the CFG is traversed. Used to estimate the execution frequency of basic blocks.
- **Value Profiling**: Collects actual runtime values for expressions, such as:
    - Indirect function call targets (for devirtualization).
    - Loop trip counts.
    - Constant values for variables.
- **Auto-FDO**: Collects profiles using hardware performance counters (e.g., `perf` on Linux). It is low-overhead, but requires debug information to map hardware samples back to the compiler IR.

## 2. Using Profile Information

Profile data guides many optimization decisions:
- **Inlining**: Prioritizes inlining of "hot" call sites.
- **Code Layout**: Reorders functions and basic blocks to maximize instruction cache locality (e.g., placing hot blocks together and cold blocks in a separate "cold" section).
- **Loop Transformations**: Unrolling factors, vectorization, and loop fission are adjusted based on loop trip counts.
- **Branch Prediction**: Static heuristics (e.g., "loops are likely to iterate", "non-null pointers") are overridden by precise runtime counts.

## 3. Profile-Guided Optimization (PGO) Workflow

1. **Instrumentation**: The compiler adds instrumentation code to the program (`-fprofile-generate`).
2. **Training**: Run the instrumented program with representative workloads.
3. **Feedback**: The compiler uses the generated profile data (`-fprofile-use`) to re-compile the program with optimizations tailored to that specific workload.

## 4. Static Profile Estimation

When runtime data is unavailable, the compiler uses heuristics to estimate branch probabilities:
- **Loop Heuristics**: Loop headers are likely executed; loop exits are likely taken.
- **No-Return Heuristics**: Calls to functions like `abort()` or `exit()` are extremely unlikely.
- **Pointer/Integer Heuristics**: Pointers are often non-null; variables used in loop bounds have predictable behavior.
- **Combining Heuristics**: Compilers use frameworks like Dempster-Shaffer or weighted combination to derive a single probability from multiple independent heuristics.

## 5. Modern Trends

- **BOLT / Propeller**: Post-link optimizers that work at the machine-code level to optimize binary layout based on `perf` data.
- **LTO Integration**: PGO data is often streamed and aggregated during LTO, allowing for whole-program profile-guided optimization.
