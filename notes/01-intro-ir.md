# Lecture 1: Introduction to Intermediate Representations and Data-Flow Analysis

This lecture covers the transition from the frontend to the middle-end of a compiler, focusing on intermediate representations (IR) and the foundational framework for data-flow analysis.

## 1. Compiler Frontend and IR Generation

The frontend is responsible for translating source code into a language-independent intermediate representation.

### Stages of the Frontend

1. **Lexical Analysis**: Breaking the source code into tokens.
2. **Parsing**: Building an **Abstract Syntax Tree (AST)** specific to the source language.
3. **IR Generation**: Lowering the AST into one or more Intermediate Languages (IL/IR).

### Intermediate Representation (IR) Types
Compilers often use multiple levels of IR to perform different types of optimizations:
- **High-Level IR**: Often tree-based (e.g., GCC's **GENERIC**, **GIMPLE**, or LLVM IR). It preserves high-level constructs like loops and array access.
- **Mid-Level IR**: More atomic types, simplified aggregates.
- **Low-Level IR**: Closer to the target architecture. GCC uses **RTL** (Register Transfer Language), which uses Lisp-like syntax and describes operations on registers and memory.

#### Machine Modes (GCC Specific)

In low-level IR like RTL, types are represented as machine modes:
- **Integer Modes**:
    - `QI`: Quarter-Integer (8-bit)
    - `HI`: Half-Integer (16-bit)
    - `SI`: Single-Integer (32-bit)
    - `DI`: Double-Integer (64-bit)
- **Floating Point Modes**: `HF` (half), `SF` (single), `DF` (double), `XF` (extended).
- **Other**: `BI` (boolean), `PI` (partial integer).

---

## 2. Symbol Table and Program Structure

The **Symbol Table** tracks all global and local entities.

### Symbols

- **Variables and Functions**: Represented by names, types, and visibility.
- **Visibility**:
  - `public`: Accessible from other translation units.
  - `local` (static): Scoped to the current unit.
  - `external`: Defined elsewhere.
  - `comdat`: Used for merging identical symbols (e.g., inline functions, template instantiations).
  - `weak`: Optional symbols that can be overridden.

### Memory Sections

Compiled symbols are placed into specific binary sections:
- `.text`: Executable code.
- `.data`: Initialized global/static variables.
- `.rodata`: Read-only data (constants).
- `.bss`: Uninitialized data (zero-filled at runtime).

---

## 3. Control Flow Graph (CFG)

The CFG is the primary data structure for middle-end optimizations.

- **Basic Block (BB)**: A maximal sequence of instructions with a single entry point (the first instruction) and a single exit point (the last instruction). No jumps into or out of the middle of the block.
- **Vertices**: Basic blocks.
- **Edges**: Possible control flow transfers (jumps, branches, fall-throughs).
- **Entry and Exit**: Special nodes representing the start and end of a function.

---

## 4. Data-Flow Analysis Framework

Data-flow analysis is a technique for gathering information about the possible set of values calculated at various points in a computer program.

### Semi-Lattice Theory

We model the information being propagated using a **semi-lattice** $(L, \wedge)$:

- **Set $L$**: The domain of values (e.g., constant values).
- **Meet Operation ($\wedge$)**: Combines information from two paths.
  - **Idempotency**: $x \wedge x = x$
  - **Commutativity**: $x \wedge y = y \wedge x$
  - **Associativity**: $(x \wedge y) \wedge z = x \wedge (y \wedge z)$

#### Lattice Elements

- **Top ($\top$)**: Undefined or "no information yet".
- **Bottom ($\perp$) or VAR**: Non-constant or "multiple conflicting values".
- **Constants**: Intermediate values (0, 1, 2, ...).

**Meet examples for Constant Propagation**:
- $c \wedge \top = c$
- $c_1 \wedge c_2 = \perp$ (if $c_1 \neq c_2$)
- $c \wedge \perp = \perp$

### Monotonicity

A function $f: L \to L$ is **monotone** if $x \leq y \implies f(x) \leq f(y)$.
In the context of data-flow, the **transfer functions** (jump functions $J_B$) must be monotone to ensure that the iterative algorithm moves in one direction through the lattice.

### Iterative Algorithm and Termination

The analysis reaches a **fixed point** by iteratively applying jump functions across the CFG.

**Theorem**: The iterative algorithm terminates if:
1. The jump functions are monotone.
2. The semi-lattice has **finite height** (no infinite descending chains).

---

## 5. Scalar Optimizations

### Constant Propagation (CP)

Goal: Replace variables that have a known constant value at compile time with that constant.

#### Data-Flow State Trace

Consider an IF-ELSE block:
1. `a = UND` ($\top$) at entry.
2. Left branch: `a = 0`. Right branch: `a = 1`.
3. At the merge point: $0 \wedge 1 = \perp$ (VAR). The compiler must assume `a` is not a constant.

#### Unreachable Code Elimination

Constant propagation can identify branches that are never taken.
- If `a = 2` and we hit `IF (a > 2)`, the "true" branch is marked unreachable ($\top$ state).
- The value of variables at the merge point will only consider the reachable paths.

### Liveness Analysis

Goal: Determine if a variable's current value will be used in the future.

**Definition**: A variable `a` is **live** at point `p` if there is a path from `p` to a **use** of `a` that does not pass through a **definition** of `a`.

- **Backward Analysis**: Liveness information flows backwards from uses to definitions.
- **Uses**: Generate liveness.
- **Definitions**: Kill liveness.
