# Lecture 2: Data-Flow Analysis and Dominators

This lecture dives deeper into the theory and implementation of data-flow analysis, covering classic bit-vector problems, convergence properties, and the foundation for Static Single Assignment (SSA) form: dominators.

## 1. Liveness Analysis Revisited

Liveness is a backward data-flow analysis used to determine which variables are "live" (will be used before being redefined) at any point in the CFG.

### Formal Equations

- **Local Live ($LocalLive_b$)**: The set of variables used in block $b$ before any definition of that same variable in $b$.
- **Kill ($Kill_b$)**: The set of variables defined in block $b$ before any use.

The liveness sets at the entry ($IN$) and exit ($OUT$) of a block $b$ are defined as:
$$LIVE\_IN(b) = LocalLive_b \cup (LIVE\_OUT(b) \setminus Kill_b)$$
$$LIVE\_OUT(b) = \bigcup_{s \in Succ(b)} LIVE\_IN(s)$$

### Implementation

Liveness is typically implemented using **bit-vectors** (bitmaps), where each bit represents a variable.
- **Forward/Backward**: Backward.
- **Meet Operation**: Union ($\cup$).

---

## 2. Iterative Data-Flow Convergence

The efficiency of iterative data-flow analysis depends on the order in which basic blocks are processed and the structure of the CFG.

### Traversal Orders

- **Postorder**: Children are visited before parents.
- **Reverse Postorder (RPO)**: Parents are visited before children (topological order for DAGs).
  - For a **DAG**, RPO ensures convergence in just **one pass** (plus one to confirm the fixed point).

### Convergence Complexity (Kam & Ullman, 1972)

For most bit-vector problems on a **reducible CFG**, the iterative algorithm terminates in:
$$\text{Iterations} \leq d(CFG) + 3$$
- **$d(CFG)$ (Loop Connectivity)**: The maximum number of back-edges on any cycle-free path in the CFG. Intuitively, this relates to the "nesting depth" of loops.

### Reducible vs. Irreducible CFG

- **Reducible CFG**: A CFG where every cycle has a single entry point (the header). Such graphs can be collapsed to a single node using $T_1$ and $T_2$ transformations.
- **Irreducible CFG**: Contains cycles with multiple entry points. This usually happens when using `goto` or specialized idioms like **Duff's Device** (a `switch` interleaved with a `for` loop).

---

## 3. Classic Bit-Vector Problems

### Reaching Definitions (RD)

Determines which definitions of a variable (at a specific instruction) might reach a point in the program.
- **Direction**: Forward.
- **Meet**: Union ($\cup$).
- **Equations**:
    $$RD\_IN(b) = \bigcup_{p \in Pred(b)} RD\_OUT(p)$$
    $$RD\_OUT(b) = RD_{local}(b) \cup (RD\_IN(b) \setminus Kill_b)$$

### Availability Analysis (AV)

Used for Common Subexpression Elimination (CSE). An expression $A + B$ is available at point $p$ if it is computed on **all** paths reaching $p$ and none of its operands have been redefined since the last computation.
- **Direction**: Forward.
- **Meet**: Intersection ($\bigcap$).
- **Equations**:
    $$AV\_IN(b) = \bigcap_{p \in Pred(b)} AV\_OUT(p)$$
    $$AV\_OUT(b) = AV_{local}(b) \cup (AV\_IN(b) \setminus Kill_b)$$

---

## 4. Dominators

Dominance is a critical concept for SSA construction and loop optimizations.

### Definition

A basic block $A$ **dominates** block $B$ ($A \text{ dom } B$) if every path from the `ENTRY` node to $B$ must go through $A$.

- **Reflexivity**: $A \text{ dom } A$.
- **Transitivity**: $(A \text{ dom } B \wedge B \text{ dom } C) \implies A \text{ dom } C$.
- **Anti-symmetry**: $(A \text{ dom } B \wedge B \text{ dom } A) \implies A = B$.

### Dominator Equations

Dominance can be solved as a forward data-flow problem:
$$Dom(ENTRY) = \{ENTRY\}$$
$$Dom(b) = \{b\} \cup \bigcap_{p \in Pred(b)} Dom(p)$$

### Dominator Tree

The **Immediate Dominator** ($IDom(B)$) is the unique node $A$ that dominates $B$, such that $A \neq B$ and $A$ does not dominate any other dominator of $B$.
The set of immediate dominance relationships forms a **Dominator Tree**.

#### Complexity

- Simple iterative solver: $O(N^2)$.
- **Lengauer-Tarjan (1979)**: $O(E \cdot \alpha(V))$ using the inverse Ackermann function. In practice, compilers use more modern fast dominance algorithms (like Cooper et al., 2004) which are easier to implement and highly efficient.
