# Lecture 2: Data-Flow Analysis and Dominators

This lecture dives deeper into the theory and implementation of data-flow analysis, covering classic bit-vector problems, convergence properties, and the foundation for Static Single Assignment (SSA) form: dominators.

## 1. Liveness Analysis Revisited

Liveness is a backward data-flow analysis used to determine which variables are "live" (will be used before being redefined) at any point in the CFG.

### Formal Equations

- **Local Live ($LocalLive_b$)**: The set of variables used in block $b$ before any definition of that same variable in $b$.
- **Kill ($Kill_b$)**: The set of variables defined in block $b$ before any use.

The liveness sets at the entry ($\text{IN}$) and exit ($\text{OUT}$) of a block $b$ are defined as:
$$\text{LIVE\_IN}(b) = LocalLive_b \cup (\text{LIVE\_OUT}(b) \setminus Kill_b)$$
$$\text{LIVE\_OUT}(b) = \bigcup_{s \in Succ(b)} \text{LIVE\_IN}(s)$$

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

## Convergence Complexity and Reducibility

### Proof Sketch: Iterative Data-Flow Convergence

The convergence is guaranteed by the **Monotonicity** of the transfer functions and the **finite height** of the semi-lattice. 

1. Let $L$ be a semi-lattice with height $h$.
2. In each iteration of the data-flow algorithm, at least one value in the semi-lattice must change (by moving down).
3. If an iteration does not produce a change, the fixed point is reached.
4. For bit-vector problems on a reducible graph, the information flow only needs to traverse loop back-edges $d(CFG)$ times, hence $d(CFG) + 3$ iterations.

### Reducible vs. Irreducible CFG

A CFG is **reducible** if it can be reduced to a single node by repeatedly applying these two transformations:
- **T1**: Remove a self-looping edge.
- **T2**: If node $n$ has only one predecessor $m$, merge $n$ into $m$.

### Reaching Definitions (RD) - Bitvector

For a function with $N$ definitions, the bitvector $V$ has $N$ bits.
- $V[i] = 1$ if definition $i$ reaches this point.
- Example: $V = 10010_2$ means definitions 1 and 4 reach this point.
- $RD_{\text{IN}}(b) = \bigvee RD_{\text{OUT}}(p)$ (bit-wise OR of predecessor vectors).

---

## 3. Classic Bit-Vector Problems

### Reaching Definitions (RD)

Determines which definitions of a variable (at a specific instruction) might reach a point in the program.
- **Direction**: Forward.
- **Meet**: Union ($\cup$).
- **Equations**:
    $$\text{RD\_IN}(b) = \bigcup_{p \in Pred(b)} \text{RD\_OUT}(p)$$
    $$\text{RD\_OUT}(b) = \text{RD}_{\text{local}}(b) \cup (\text{RD\_IN}(b) \setminus Kill_b)$$

### Availability Analysis (AV)

Used for Common Subexpression Elimination (CSE). An expression $A + B$ is available at point $p$ if it is computed on **all** paths reaching $p$ and none of its operands have been redefined since the last computation.
- **Direction**: Forward.
- **Meet**: Intersection ($\bigcap$).
- **Equations**:
    $$\text{AV\_IN}(b) = \bigcap_{p \in Pred(b)} \text{AV\_OUT}(p)$$
    $$\text{AV\_OUT}(b) = \text{AV}_{\text{local}}(b) \cup (\text{AV\_IN}(b) \setminus Kill_b)$$

---

## 4. Dominators

Dominance is a critical concept for SSA construction and loop optimizations.

### Definition

A basic block $A$ **dominates** block $B$ ($A \text{ dom } B$) if every path from the $\text{ENTRY}$ node to $B$ must go through $A$.

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

### Strict Dominance

Node $A$ **strictly dominates** node $B$ ($A \text{ sdom } B$) if $A$ dominates $B$ and $A \neq B$.