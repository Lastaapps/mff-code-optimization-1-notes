# Glossary of Compiler Optimization Terms

This glossary provides concise definitions of key concepts covered in the course.

- **Alias Analysis**: The process of determining whether two memory references (e.g., pointers) refer to the same location.
- **Anticipancy (Down-safety)**: A property of an expression indicating it is calculated on all paths from a program point to the exit before its operands are modified.
- **Availability**: A property of an expression indicating it has already been computed on all paths reaching a program point, and its operands have not been modified.
- **Back-edge**: A CFG edge from a node to one of its ancestors in a Depth-First Search (DFS) tree, typically used to identify loop back-paths.
- **Basic Block (BB)**: A maximal sequence of instructions with one entry point and one exit point.
- **Control Dependence**: A property where a basic block $C$ is control dependent on $B$ if $B$ determines whether $C$ will be executed (e.g., an `IF` statement).
- **Critical Edge**: A CFG edge from a block with multiple successors to a block with multiple predecessors.
- **Dominance**: Block $A$ dominates block $B$ if every path from the entry to $B$ must pass through $A$.
- **Dominance Frontier (DF)**: The set of nodes where dominance "ends," identifying merge points where $\phi$ functions are needed.
- **Factored Use-Def (FUD) Chains**: A historical representation that linked uses to all possible definitions, a precursor to SSA form.
- **Identical Code Folding (ICF)**: An interprocedural optimization that merges functions with identical bodies.
- **Intermediate Representation (IR)**: The internal language used by a compiler to represent the program after frontend parsing and before code generation.
- **Lazy Code Motion (LCM)**: An optimal Partial Redundancy Elimination (PRE) algorithm that minimizes both redundant computations and register pressure.
- **Liveness Analysis**: A data-flow analysis that determines which variables are "live" (will be used in the future) at any given program point.
- **Loop Invariant Code Motion (LICM)**: An optimization that hoists computations out of a loop if the operands remain constant throughout the loop.
- **Partial Redundancy Elimination (PRE)**: An optimization that eliminates redundant computations even if they are only redundant on some paths through the CFG.
- **Points-To Analysis (PTA)**: A form of alias analysis that computes the set of memory locations a pointer can refer to.
- **Post-Dominance**: The dual of dominance: $B$ post-dominates $C$ if every path from $C$ to the exit must pass through $B$.
- **Profile-Guided Optimization (PGO)**: A technique that uses runtime execution data (profiles) to guide compiler optimizations like inlining and code layout.
- **Reducible CFG**: A control flow graph where every cycle has a single entry point (the header).
- **Sparse Conditional Constant Propagation (SCCP)**: An optimization that propagates constants while simultaneously removing unreachable code.
- **Static Single Assignment (SSA)**: A representation where every variable is assigned exactly once.
- **ThinLTO**: A scalable version of Link-Time Optimization that uses function summaries and distributed backends to handle large codebases.
- **Type-Based Alias Analysis (TBAA)**: Alias analysis based on the assumption that pointers of different types (with specific exceptions) do not alias.
- **Value Numbering (VN)**: An optimization that assigns numeric identifiers to expressions to identify redundant computations regardless of variable names.
