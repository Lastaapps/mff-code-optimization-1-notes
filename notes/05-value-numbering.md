# Lecture 5: Value Numbering and SSA Optimizations

This lecture covers optimizations that leverage the properties of SSA form to perform efficient global analysis, specifically focusing on Sparse Conditional Constant Propagation (SCCP) and Value Numbering.

## 1. Sparse Conditional Constant Propagation (SCCP)

SCCP (Wegman & Zadeck, 1991) is a highly effective optimization that combines constant propagation with unreachable code elimination in a single pass over the SSA graph.

### The Algorithm

SCCP uses two worklists to track changes:

1. **Flow Worklist**: Tracks edges in the CFG that are "executable". Initially, only the edge from $\text{ENTRY}$ is executable.
2. **SSA Worklist**: Tracks SSA names whose lattice values have changed and whose uses need re-evaluation.

#### Logic:

- When an edge $e \to B$ becomes executable:
  - Visit all $\phi$ functions in $B$.
  - If $B$ is visited for the first time, visit all statements in $B$.
- When an SSA name $n$ changes its lattice value:
  - Re-evaluate all statements that use $n$, provided the block containing the statement is reachable via an executable edge.
- **Control Flow**: If a branch instruction evaluates to a constant, only the corresponding edge is added to the Flow Worklist. If it evaluates to $\perp$, all outgoing edges are added.

### Use Case: Branch Prediction and Dead Code Elimination
```
if (x > 10) { ... } // If constant propagation shows x=5, this entire block is dead.
```
SCCP identifies that the condition is always false, removes the block, and then propagates the constant `x=5` to the next block, potentially enabling further constant folding.

---

## 2. Global Value Numbering (GVN)

GVN aims to identify redundant computations by assigning a **Value Number** ($\text{VN}$) to each expression. If $\text{VN}(x) = \text{VN}(y)$, then $x$ and $y$ are guaranteed to have the same value at runtime.

### Congruence-Based GVN

- Treats the problem as a partition refinement.
- Initially, all variables with the same operator and type are in the same partition.
- Partitions are iteratively split if their operands belong to different partitions.

### RPO-GVN

- An iterative approach that processes blocks in **Reverse Postorder**.
- Uses a hash table to store expression values discovered across the function.
- Special handling for $\phi$ functions is required to maintain consistency across merges.

### Use Case: Algebraic Identities

GVN can detect that `x + y` and `y + x` have the same value number because the operator `+` is commutative.

Original:
```
t1 = a + b
...
t2 = b + a
```
Optimized by GVN:
```
t1 = a + b
...
t2 = t1
```
The second expression is replaced by the first, allowing subsequent copy propagation to replace `t2` with `t1`.

---

## 3. Local Value Numbering (LVN)

- Performed within a single basic block.
- Uses a hash table of expressions (e.g., `plus, VN(a), VN(b)`).
- If an expression is already in the table, replace it with the previously computed value.

---

## 4. Comparison: CSE vs. GVN

- **Common Subexpression Elimination (CSE)**: Lexical-based. It finds expressions that are identical in text (e.g., `a + b` and `a + b`).
- **Value Numbering (VN)**: Algebraic and semantic-based. It can identify that `x = a + b` and `y = b + a` are the same (using commutativity) or that different variable names represent the same value if they have the same value number.
- **SSA + GVN**: In SSA form, GVN becomes much simpler because variable names are unique and values do not change after assignment.
