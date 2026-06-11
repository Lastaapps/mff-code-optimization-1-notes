# Lecture 4: Static Single Assignment (SSA) Form

Static Single Assignment (SSA) is an intermediate representation where every variable is assigned exactly once. This property simplifies many data-flow analyses and optimizations.

## 1. SSA Definition and Motivation

A Control Flow Graph (CFG) is in **SSA form** if:
1. Each variable has exactly one definition in the program text.
2. Each use of a variable refers to exactly one definition.

### The $\phi$ (Phi) Function

To handle points where different control flow paths merge, SSA introduces $\phi$ functions.
- **Example**:
    ```
    if (cond) {
        x1 = 0;
    } else {
        x2 = 1;
    }
    x3 = phi(x1, x2);
    ```
- The $\phi$ function "chooses" the value based on which edge was used to reach the block.

---

## 2. SSA Construction Algorithm (Cytron et al., 1989)

The standard construction algorithm consists of four main steps:

### Step 1: Computing Dominators
Compilers typically use the **Cooper, Harvey, and Kennedy (2004)** algorithm or **Lengauer-Tarjan** for this.

- **Cooper's Algorithm**: An iterative solver using a "2-finger" intersection algorithm on dominator sets represented as RPO-ordered lists.
  - Complexity: $O(N \cdot D)$, where $D$ is the loop nesting depth.

### Step 2: Computing Dominance Frontiers (DF)

The **Dominance Frontier** of a block $B$ is the set of all nodes $C$ such that $B$ dominates a predecessor of $C$, but $B$ does not strictly dominate $C$.
$$DF(B) = \{C \mid (\exists p \in Pred(C)) [B \text{ dom } p \wedge \neg(B \text{ sdom } C)]\}$$

### Step 3: $\phi$ Placement

For a variable $V$ defined in a set of blocks $X$:
1. Identify the **Iterated Dominance Frontier** $DF^+(X)$.
2. Place a $\phi$ function for $V$ in every block belonging to $DF^+(X)$.
3. This ensures that a $\phi$ node exists at every "join point" where multiple definitions of $V$ could reach.

### Step 4: Renaming

After $\phi$ placement, variables are renamed (e.g., $x \to x_1, x_2, \dots$).
- The algorithm traverses the **Dominator Tree** in Depth-First Search (DFS) order.
- It maintains a stack of current versions for each variable.
- For each block:
  1. Rename definitions and uses in the block using the stack.
  2. Update $\phi$ function operands in successor blocks.
  3. Recursively visit children in the dominator tree.
  4. Pop the stack to restore the state before moving to a sibling.

---

## 3. SSA Variants

- **Minimal SSA**: The standard form produced by the Cytron algorithm.
- **Pruned SSA**: Reduces the number of $\phi$ functions by only placing them for variables that are **live** at the join point. This requires a liveness analysis pass before SSA construction.
- **Semi-Pruned SSA**: A compromise that only places $\phi$ functions for variables that are live across at least one basic block boundary (non-local variables).
