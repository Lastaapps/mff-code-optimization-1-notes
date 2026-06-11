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

The standard construction algorithm consists of four main steps. **See slides for more complete explanation**.

## 2. SSA Construction Algorithm (Cytron et al., 1989)

### Step 1: Computing Dominators (The "2-Finger" Intersection)

When computing dominators iteratively, the dominator sets are represented as ordered linked lists (based on RPO).

- **The Intersection Algorithm**:
  Two pointers ("fingers") traverse two dominator lists $D(p_1)$ and $D(p_2)$ backwards (from exit to entry).
  - If the pointers refer to the same node $n$, then $n$ is in the intersection.
  - If one pointer refers to a node $n$ and the other to $m$, increment the pointer pointing to the node with the smaller DFS number (the one "higher" in the dominator tree).
  - Repeat until the pointers meet.

### Step 2: Computing Dominance Frontiers (DF)

The **Dominance Frontier** of a block $B$ is the set of all nodes $C$ such that $B$ dominates a predecessor of $C$, but $B$ does not strictly dominate $C$.

$$DF(B) = \{C \mid (\exists p \in Pred(C)) [B \text{ dom } p \wedge \neg(B \text{ sdom } C)]\}$$

### Step 3: $\phi$ Placement

For a variable $V$ defined in a set of blocks $X$:
1. Identify the **Iterated Dominance Frontier** $DF^+(X)$.
2. Place a $\phi$ function for $V$ in every block belonging to $DF^+(X)$.
3. This ensures that a $\phi$ node exists at every "join point" where multiple definitions of $V$ could reach.

#### Step 2 & 3: Dominance Frontiers and $\phi$ Placement Example
CFG:
```
      [Entry]
         |
        [1] <----+
       /   \     |
     [2]   [3] --+
       \   /
        [4]
```
1. **Dominators**: $IDom(1)=Entry, IDom(2)=1, IDom(3)=1, IDom(4)=1$.
2. **DF(1)**: Successors are {2, 3}. 1 dominates {2, 3}, so $DF(1) = \emptyset$.
3. **DF(2)**: Successor is {4}. 2 does not dominate 4 (path via 3 does not contain 2), so $DF(2)=\{4\}$.
4. **DF(3)**: Successor is {4}. 3 does not dominate 4, so $DF(3)=\{4\}$.
5. **$\phi$ placement for $X$**: If $X$ is defined in blocks {2, 3}, then $DF^+(X) = \{4\}$. We place $\phi$ at the entry of block 4: `X = phi(X2, X3)`.

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
