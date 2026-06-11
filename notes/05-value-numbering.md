# Lecture 5: Value Numbering and SSA Optimizations

This lecture covers optimizations that leverage the properties of SSA form to perform efficient global analysis, specifically focusing on Sparse Conditional Constant Propagation (SCCP) and Value Numbering.

## 1. Sparse Conditional Constant Propagation (SCCP)

SCCP (Wegman & Zadeck, 1991) is a highly effective optimization that combines constant propagation with unreachable code elimination in a single pass over the SSA graph.

### The SCCP Lattice

Each SSA name is assigned a lattice value:
- **Top ($\top$)**: Undefined or unknown. The initial state for all variables.
- **Constant ($c$)**: The variable has a specific constant value.
- **Bottom ($\perp$) or VARYING**: The variable is known to be non-constant.

### The Algorithm

SCCP uses two worklists to track changes:

1. **Flow Worklist**: Tracks edges in the CFG that are "executable". Initially, only the edge from `ENTRY` is executable.
2. **SSA Worklist**: Tracks SSA names whose lattice values have changed and whose uses need re-evaluation.

#### Logic:
- When an edge $e \to B$ becomes executable:
  - Visit all $\phi$ functions in $B$.
  - If $B$ is visited for the first time, visit all statements in $B$.
- When an SSA name $n$ changes its lattice value:
  - Re-evaluate all statements that use $n$, provided the block containing the statement is reachable via an executable edge.
- **Control Flow**: If a branch instruction evaluates to a constant, only the corresponding edge is added to the Flow Worklist. If it evaluates to $\perp$, all outgoing edges are added.

---

## 2. Global Value Numbering (GVN)

GVN aims to identify redundant computations by assigning a **Value Number** ($VN$) to each expression. If $VN(x) = VN(y)$, then $x$ and $y$ are guaranteed to have the same value at runtime.

### Local Value Numbering (LVN)

- Performed within a single basic block.
- Uses a hash table of expressions (e.g., `plus, VN(a), VN(b)`).
- If an expression is already in the table, replace it with the previously computed value.

### Congruence-Based GVN

- Treats the problem as a partition refinement.
- Initially, all variables with the same operator and type are in the same partition.
- Partitions are iteratively split if their operands belong to different partitions.
- This approach is powerful but can be complex to implement efficiently.

### RPO-GVN

- An iterative approach that processes blocks in **Reverse Postorder**.
- Uses a hash table to store expression values discovered across the function.
- Special handling for $\phi$ functions is required to maintain consistency across merges.

---

## 3. Comparison: CSE vs. GVN

- **Common Subexpression Elimination (CSE)**: Lexical-based. It finds expressions that are identical in text (e.g., `a + b` and `a + b`).
- **Value Numbering (VN)**: Algebraic and semantic-based. It can identify that `x = a + b` and `y = b + a` are the same (using commutativity) or that different variable names represent the same value if they have the same value number.
- **SSA + GVN**: In SSA form, GVN becomes much simpler because variable names are unique and values do not change after assignment.
