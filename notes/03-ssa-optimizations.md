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

LCM (Knoop et al., 1992) is the state-of-the-art algorithm for PRE. It improves upon earlier methods by being "lazy" — it hoists code only as far as necessary, thereby minimizing the live ranges of registers and reducing register pressure.

### The LCM Algorithm Steps

LCM is solved using a series of four bit-vector data-flow analyses:

1. **Anticipancy (Down-safety)**: A backward analysis. An expression is anticipated at point $p$ if it is computed on all paths from $p$ to `EXIT` before any of its operands are modified.
   - $ANT\_IN(b) = EUSE_b \cup (ANT\_OUT(b) \setminus Kill_b)$
   - $ANT\_OUT(b) = \bigcap_{s \in Succ(b)} ANT\_IN(s)$
2. **Availability**: A forward analysis (same as in CSE).
3. **Earliest Placement**:
    $$EARLIEST(i, j) = ANT\_IN(j) \cap \neg AV\_OUT(i) \cap (Kill_i \cup \neg ANT\_OUT(i))$$
    This identifies the earliest points where an expression can be safely moved.
4. **Postponability**: A forward analysis that determines if a computation at an `EARLIEST` point can be delayed further without losing redundancy elimination benefits.
5. **Latest Placement**: The final points where the expression is inserted.

---

## 4. Other Scalar Optimizations

- **Conditional Constant Propagation (CCP)**: Propagates constants through the CFG while simultaneously identifying and removing unreachable branches.
- **Copy Propagation**: Replaces uses of a variable $B$ with $A$ if there is an assignment $B = A$ and neither has been modified.
- **Dead Code Elimination (DCE)**: Removes instructions that have no side effects and whose results are never used.
