# Lecture 7: Interprocedural Optimization (IPO), Inlining, and LTO

This lecture explores interprocedural optimizations that operate across function boundaries, specifically focusing on the evolution of GCC's inliner and modern Link-Time Optimization (LTO) techniques.

## 1. Inlining Strategy

Inlining is a key enabler for other optimizations, as it exposes the bodies of callee functions to callers.

- **Evolution**:
    - **Bottom-Up**: Traditional approach within a single translation unit.
    - **Modern GCC (2003+)**: Uses a **Greedy Inliner**. It parses the whole unit, builds a **Callgraph (CGRAPH)**, and greedily marks edges for inlining based on a `speed/size` cost-benefit heuristic.
- **Attributes**:
    - `[[always_inline]]`: Forces inlining, bypassing cost heuristics.
    - `[[flatten]]` / `[[leafify]]`: Forces inlining of all calls within a function, effectively flattening the call sub-tree.
- **Thresholds**: Compilers use thresholds (e.g., auto-inlining limits, large unit growth limits of ~200%) to balance binary size with execution performance.

## 2. Link-Time Optimization (LTO)

LTO defers optimization until link time, allowing the compiler to optimize across separate translation units.

- **Traditional LTO**: Requires all Intermediate Representation (IR) to be resident in memory during the Whole Program Analysis (WPA) phase. This does not scale to extremely large projects (e.g., browsers or large server-side applications).
- **ThinLTO (Xinliang David Li, Google)**: A scalable, distributed alternative:
    - **Summaries**: Each translation unit generates a small summary (function signatures, call graphs, profile data).
    - **Thin Linking**: The linker only links these summaries, rather than the full IR.
    - **Distributed Backend**: Each translation unit is optimized in parallel. When inlining is decided (based on the global summary), the required IR is imported from the original unit.

## 3. Other Interprocedural Techniques

- **Identical Code Folding (ICF)**: Merges functions with identical bodies to reduce code size.
- **Interprocedural Constant Propagation (IPA-CP)**: Clones functions based on specific call contexts (e.g., constant arguments) and optimizes each clone independently.
- **Devirtualization**: Uses alias/points-to analysis to determine the actual target of a virtual function call, allowing the call to be inlined or replaced with a direct jump.
