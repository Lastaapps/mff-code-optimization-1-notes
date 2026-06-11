# Lecture 6: Alias Analysis

Alias Analysis is the process of determining whether two pointers (or memory references) refer to the same memory location. It is essential for enabling optimizations like Load/Store elimination and Loop transformations.

## 1. Categorization

- **NO ALIAS**: Two pointers definitely point to different memory locations. (Great for optimizations)
- **MAY ALIAS**: Two pointers might point to the same memory location. (Conservative assumption)
- **MUST ALIAS**: Two pointers definitely point to the same memory location.

## 2. Alias Oracles

An alias oracle is a component that answers the query: "Do memory access A and access B alias?"

### Key Alias Analysis Techniques:

1. **Type-Based Alias Analysis (TBAA)**: Based on the C/C++ language rules (e.g., `float*` and `int*` are assumed not to alias, except for `char*` or `void*`).
2. **Points-To Analysis (PTA)**: Computes the set of memory locations $PT(p)$ that a pointer $p$ can point to. If $PT(p) \cap PT(q) = \emptyset$, then $p$ and $q$ do not alias.
3. **Access Path Oracle**: Uses the structure of memory access (e.g., `p->foo.bar` and `q->bar`) to determine overlap.
4. **MOD/REF Analysis**: Tracks the set of memory locations modified or referenced by a function call.

## 3. Function Attributes for Alias Analysis

Modern compilers allow functions to be annotated with attributes that improve alias analysis:

- **NO CAPTURE**: The pointer argument does not escape the function scope.
- **NO CLOBBER**: The function does not modify the memory pointed to by the argument.
- **NO READ**: The function does not read the memory pointed to by the argument.
- **NO ESCAPE**: The pointer does not escape (often implies NO CAPTURE).

## 4. Why Alias Analysis Matters

Alias information is required for:
- **Redundant Load/Store Elimination**: Knowing that a store does not alias with a subsequent load allows the reuse of the loaded value.
- **Store Sinking**: Moving memory operations out of loops if the analysis proves no interference.
- **Loop Optimizations**: Vectorization and software pipelining depend on knowing that vector indices do not overlap (or are independent).

---

## 5. Summary of Related Analyses

- **MOD/REF Analysis**: Provides fine-grained information about memory modifications per function call.
- **Project Ranger (GCC)**: A demand-driven range engine that can integrate bitwise information and alias analysis facts to refine memory access bounds.
- **Control Dependence**: Essential for aggressive Dead Code Elimination (CD-DCE). It ensures that control flow leading to live statements is preserved, even if the branch itself is not directly data-dependent on a live value.
