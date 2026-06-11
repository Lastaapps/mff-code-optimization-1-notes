# Lecture 3: Code Motion and SSA Optimizations

This lecture covers advanced scalar optimizations that move code to eliminate redundancy, with a focus on Partial Redundancy Elimination (PRE) and Lazy Code Motion (LCM).

## 1. Loop Invariant Code Motion (LICM)

LICM hoists (vynese, vytáhne) computations out of a loop body if their results do not change during the loop's execution.

- **Pre-header**: A synthetic basic block inserted immediately before the loop header to serve as a landing pad for hoisted code.
- **Condition for Hoisting**: An expression $A + B$ can be hoisted to the pre-header if:
   1. Both $A$ and $B$ are loop-invariant (not modified within the loop).
   2. The placement is safe (it won't introduce a fault that wouldn't have occurred in the original program).

---

## 2. Partial Redundancy Elimination (PRE)

PRE is a general optimization that subsumes both Common Subexpression Elimination (CSE) and Loop Invariant Code Motion (LICM).

- **Definition**: An expression is **partially redundant** if it is computed more than once on *some* paths through the CFG, but not necessarily all.
- **Goal**: Insert computations on paths where the expression is missing to make it **fully redundant**, then eliminate the original redundant computations.

### Critical Edge Splitting

A critical edge is an edge from a block with multiple successors to a block with multiple predecessors.
**Splitting critical edges** is a mandatory transformation before performing PRE or LCM. It ensures there is a dedicated basic block to hold hoisted computations without affecting unrelated control flow paths.

---

## 3. Lazy Code Motion (LCM)

LCM (Knoop et al., 1992) is the state-of-the-art algorithm for PRE. It improves upon earlier methods by being "lazy" — it hoists code only as far as necessary, thereby minimizing the live ranges of registers.

### The LCM Algorithm Steps

LCM is solved using a series of four bit-vector data-flow analyses:

1. **Anticipancy (Down-safety) (Backward)**:
   - $\text{EUSE}_b$ (Upward Exposed Uses): Expressions used in block $b$ before any assignment to their operands in $b$.
   - $\text{ANT\_IN}(b) = \text{EUSE}_b \cup (\text{ANT\_OUT}(b) \setminus Kill_b)$
   - $\text{ANT\_OUT}(b) = \bigcap_{s \in Succ(b)} \text{ANT\_IN}(s)$
2. **Availability (Forward)**: A forward analysis.
3. **Earliest Placement**: Identifies the earliest points where an expression can be safely moved.
4. **Postponability (Forward)**: Determines if an $\text{EARLIEST}$ computation can be delayed further.
5. **Latest Placement**: Final insertion points.

---

## 4. Other Scalar Optimizations

- **Conditional Constant Propagation (CCP)**: Propagates constants through the CFG.
- **Copy Propagation**: Replaces uses of a variable $B$ with $A$ if $B = A$.
- **Dead Code Elimination (DCE)**: Removes unused instructions.
