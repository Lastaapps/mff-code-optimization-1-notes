***Generic (tree), GIMPLE (3AC)***
GCC high level languages

***WHIRL***
OPEN64

***RTL***
Register Transfer Language
GCC low level language, representation of assembler as  SEXPRs

## Intermediate Langage
*Types*
ints:
- QI 8b (quad)
- HI 16b (half)
- SI 32b (single)
- DI 64b
floats
- HF
- SF
- DF
- XF (historic 80b float)
float vectors (V16QI)
BI - boolean integer
PI - partial integer

*Symbol table*
(variables, function)
- name
- visibility
    - static, extern -> public, local (static), extern
- comdat
    - merging symbol of the same name
    - (C++ inline functions)
- weak
    - non-defined symbols
- sections
    - text, data, rodata

*Constants*
also includes constructors
big arrays - many elements, even for sparse arrays like
```c
int a[10000] = { [100] = 1 };
```
(it is zeroed out)

*Function*
declaration
body - CFG, assignments
- arithmetics - tripples, tupples with mask
- control flow - CFG
- memory - loads, stores, moves
    - `a = b.c[e]` high-level semantics and low-level operations
- function calls
- exception handling
- annotations
- debug info

## Side effects
- calls - const, pure
- volatile mem. access
- control flow
- exception handling
- sometimes memory access
- sometimes FP arithmetics

## Optimiziations
local, global, inter-procedural
function local, simple types, no address taken

*Constant propagation (CP)*
*Constant folding*

*Unreachable code removal*
*CFG simplification*
- *Edge forwarding* - jump over empty block
- *Removal of unnecessary conditionals*

*Common subexpression elimination (CSE)*
*Copy propagation (CPROP)*
```c
b = a
c = b
d = c
```

*Dead code removal* - no side effects

***Superblock*** - sequence of blocks where each has only one incoming edge
***Extended basic block*** - tree expanding downwards
***Hyperblock*** - predicated statements, IMPACT 


## Dataflow Analysis
on CFG
- Semi-lattice of values with meet
- Jump functions
- Forward/Backward/Bi-Directional

*Kim & Ulchman 72*
For reducible CFG, iterative dataflow will termine $d(CFG) + 3$ iterations
$d(g)$ - good connectivity - maximal number of backward edges in path
*reducible* - structured control flow, backward edge could be merged into a BB

*Duff's device* - example of non-reducible

*repr* - (priority) queue of BB, 

*constant propagation*
*liveness*

$LocalLive(b)$ - set of all variables used before defined
$Kill(b)$ -  set of all variables defined and not used before

DU/UD chain - definition -> use

*Reaching definitions*
$RD_{in}(b)$
- $\emptyset$ entry
- $\cup_{p \in Pred(b)} RD_{out}(p)$
$RD_{out}(b) = RD_{local}(b) \cup (RD_{in}(b) / Kill(b))$

*Common Subexpression Availability*
*Partial Redundancy Elimination*

## Dominators
BB dominates another BB if every path to the second one goes thru the first one 

$dom(entry) = entry$
$dom(b) = \set{b} \cup \cap_{p \in pred(b)} dom(p)$

*properties*
- $\forall A . Entry\ \operatorname{dom} A$
- reflexivity
- transitivity
- $A, A' \operatorname{dom} B \Rightarrow (A' \operatorname{dom} A) \lor (A \operatorname{dom} A')$ 

forms a tree
can be expressed with intervals

## SSA
'89, Cytron, Ferrante, Rosen, Wegman, Zadeck

- Computing dominators
    - Lengauer, Tarjan, $O((n + m) \alpha(m))$, $\alpha$ - inverse Ackerman
- Computing dominance frontiers
- $\phi$ placement
- renaming

*A simple fast dominance algorithm*
'04, Cooper, Harvey, Kennedy
represent dominance sets as linked lists ordered by RPO (reverse post-order)

from
```
forall B in BB
    dom(B) = V(CFG)
    
dom(entry) = entry

changed = True
while changed
    changed = False
    foreach B in RPO
        N = dom(P), P some predecesor of B
        foreach P in pred(B)
            N = intersect(dom(P), N)
        if N != dom(B)
            dom(B) = N
            changed = True 
```
to
```
forall B in BB
    dom(B) = Top (undef)

dom(entry) = {}

changed = True
while changed
    changed = False
    foreach B in RPO
        N = dom(P), P some processed predecesor of B
        foreach P in pred(B)
            N = intersect(dom(P), N)
        if N != dom(B)
            dom(B) = N
            changed = True 
```


*Dominance frontier*
$DF(B) = \set{C \mid \exists P \in pred(C). B \operatorname{dom} P \land \lnot B \operatorname{sdom} C}$
$\operatorname{sdom}$ - strictly dominates

BBs in dominance frontier need to (possibly) start with Phi nodes

DFS on dominance tree
$DF_{local}(B) = \set{C \mid C \in \operatorname{succ}(B) \land \lnot B \operatorname{sdom} C}$
$DF_{up}(B) = \set{C \in DF(B) \mid \lnot dom(B) \operatorname{sdom} C }$
$DF(B) = DF_{local}(B) \cup \cup_{z \in children(B)} DF_{up}(z)$

*Join sets*
$X \subseteq V(CFG)$
$J(X) = \set{C \in V(CFG) \mid \exists A, B \in \text{ path } p: A \rightarrow C, q: B \rightarrow X \text{ s.t. } C \text{ is first common BB on } p, q}$
$X$ - set of all definitions of var A
$J(X)$ - set of basic blocks where Phi is needed

*Iterated join*
$J_1 = J(X)$
$J_2 = J(X \cup J_1(X))$
$J^+ = \lim_{1 \rightarrow \inf}J_1$

*Iterated dominance frontier*
$DF(X) = \set{DF(B) \mid B \in X}$

$DF_1(X) = DF(X)$
$DF_2(X) = DF(X \cup DF_1(X))$
$DF^+(X) = \lim_{1 \rightarrow \inf} DF_1(X)$

*Phi placement*
for every var V
$X$ = set of al BB defining V (including entry)
insert $X = \phi(X \dots X)$ on begining of every $B \in DF^+(X)$

**theorem** If X contains entry, $J^+(X) = DF^+(X)$
*obsv* $DF^+(S) \subseteq J^+(S)$
*lemma* $DF^+(S) \supseteq J^+(S)$
*obsv*
if $X \neq Z$ BBs
a $P: X \rightarrow Z$  is a path
$\exists X' \in \set{X} \cup DF^+(X)$ on $P$ dominating $Z$
1) $X \operatorname{dom} Z$, $X = X'$
2) find $X_1$ last on $P \cap DF(X)$, ...

*obsv*
if $X \neq Y$ are BBs
paths $X \rightarrow Z$, $Y \rightarrow Z$ converging in $Z$
then $Z \in DF^+(X) \cup DF^+(Y)$
1) we have $X'$, $Y'$ from previous observation
2) $X'$ dominates $Y'$ -> 

Minimal SSA Form
Pruned SSA Form

## SCCP
sparse conditional constant propagation
Wegman, Zadeck - Constant Propagation with Conditional Branches '92
sparse dataflow - on SSA graph

```
FlowWL <- Succ(Entry)
SSAWL <- empty
forall e in E(CFG): exec(e) = False
forall n in ssa_name: lattice(n) = Top

while nonempty FlowWL or nonempty SSAWL:
    if nonempty FlowWL:
        e <- pop FlowWL
        if exec(e) = False:
            exec(e) <- True
            forall Phi in e->Dest: visitPhi(Phi)
            if e->Dest visited first time:
                forall stmt in e->Dest: visitStmt(stmt)
    if nonempty SSAWL:
        stmt <- pop SSAWL
        if stmt is Phi:
            visitPhi(stmt)
        else:
            visitStmt(stmt)
```

on $\phi$ nodes only consider the edges which are executable

## Alias analysis
true dependency - write + read
anti dependency - read + write
out dependncy - write + write

### Oracles
1. base + offset

- base pointer, offset range, size range
- no overlap, must overlap

1. type-based alias analysis (1984)

```c
float a = val;
int* p = (int*)&a;
*p = (*p)&(~sign);
return a;
```
view convert - this is incorrect, should use memcpy

unique int id for each type
in K&R `char* malloc(...)`

because of placement new, aliasing can happen

3. access path oracle

if 
```c
p->foo.bar;
q->bar;
```
are aliased, because `q` could be
```c
*q = &(p->foo);
```

4. points-to analysis
set of all locations (variables with address taken or heap locations) `p` can point to
$PT(p) \cap PT(q) = \emptyset$ => no overlap

a) constraint generation

| code              | constrain                                      |     |
| ----------------- | ---------------------------------------------- | --- |
| `p = &a`          | $\text{\&a} \in PT(p)$                         |     |
| `p = q`           | $PT(p) \supseteq PT(q)$                        |     |
| `p = *q`          | $PT(p) \supseteq \set{PT(O) \mid o \in PT(q)}$ |     |
| `p = malloc(...)` | $ Heap(lineno) \in PT(p)$                      |     |

b) constraint solving
explodes, to prevent:
- contraction of cliques `p = q` and `q = p`
- iterative solving

5. inter-procedural points-to analysis

*Steensgaard algorithm*
on `p = q` also assume `q = p` -> equailites -> union-find

*Bloom filter*

## Local Value Numbering
in one BB
*expression hash*
```
for X : definition in BB
    n = lookup(X.op, X[0], X[1])
    if n not new => replacement
    VN[X.dest] = n
```

## Global Value Numbering
in one function
for each SSA name n, VN(n)  is *value number* s.t. $VN(n) = VN(n') \Rightarrow n \text{ and } n'$ will always be the same

*Congurence based VN*
1. subdivide by types, op, constants
2. for each equality class, check if operands are equal and subdivide 

*RPO - GVN* 
```
forall n : SSA name
    VN[n] = Top

until stabilized
    forall B : BB in reverse postorder
        forall B : definition in B
            VN[X.dest] = lookup(X.op, X[0], X[1], ...)
    delete hashtable
```

*Example:*
```
BB0: (start)
    I1 = 0
    J1 = 0
    K1 = I + 1
    jmp BB1
BB1:
    I2 = Phi(I1, I3)
    J2 = Phi(J1, J3)
    I3 = I2 + 1
    J3 = J2 + 1
    jmpIf I < 10, BB1, BB2
BB2:
    exit
```
it will figure that I and J have the same value

*Conditional Execution Predication*
```
if A == 1 {
} else {
}

B = Phi(0, 1)
if A == 1 {
} else {
}
C = Phi(0, 1)
```
B and C are the same
`B = (A = 1) ? 1 : 0`

*1 GVN*

*GVN-PRE*

## Alias oralce 
- GVN -> eliminate redundant loads
- dead store removal
- Store sinking

base + Offset
PTA
TBAA
Modref (Modified & referenced)
- TBAA access tree
- loads/stores relative to parameters
- PTA Flags
    - no (indirect) capture
    - no (indirect) clobber
    - no (indirect) read
    - no (indirect) return


## Profiling
static profile estimation, profile feedback 
loop optimizations, inlining, register allocation, function layout
### Edge profiling
expected frequency of executions of BB, edges

Ball & Larus - as flow graph (toky v sítích)
- fake edge exit -> entry
- calculate minimum spanning tree (mst)
- insert counters to non-tree edges

different edges are differently expensive - change the costs from mst

HHVM - Facebook JIT

### Value profiling
indirect call
constants

### Loop histograms

### Auto-FDO
`perf` - low-overhead, not as precise (sampled)
needs to be compiled with debug info

### Binary optimizers
Bolt (Facebook)
Propeller (Google)

### Static profile estimation
*Branch prediciton for free*

- apply heuristics predicting branch probabilities
- combine
- propagate to BB frequencies

*Heuristics*
Loop:
loop iterations
guessep iterations
loop exit

Non-loop:
values are pos
noreturn

*Combining*
First batch - ordered by reliability, take the first one hitting
Dempster-Shaffer - combination of probabilities
$$
\frac{P_1 P_2}{P_1 P_2 + (1 - P_1)(1 - P_2)}
$$
not correct - $P_1$, $P_2$ should be independent

SREAL - GCC floating point 

### Profile info quality
Unknown
Guessed
AutoFDO
Adjusted
Precise

## Interprocedural Optimization
### Inlining Pointer
top-down / bottom-up

*1987-2002*
top-down
if function call is known - inline it
if function is inline (or < 100 instructions) - remember body

bottom-down
if |function| < 20 remeber
inline all knwon calls

bubbles of inlined functions

*2003*
Greedy inlner
1) parse whole file
2) build callgraph
3) greedily mark edges inline
priority queue

attribute `leafify` (`flatten`)

Early inliner - inline very small functions

*Early opts*
lower
CFG
early inliner
CCP, FRE, DCE
*Small IPA*
Profile
*IPA*
Unreachable
Inliner
Constant propagation
Attributes (pure, const, nothrow)
*Late opt*
CCP
PRE
...

## LTO
https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html
2005+
after small IPA, it was link time optimizations
(LTO streaming - serialization of data for LTO)

*WHOPR* - format, "memory dump"
WPA - whole program analysis
LTRAN - local translator

LTO plugin into linker (`LTO1`)

## Xinliang David Li
### LIPO
Lightweight Interprocedural Optimization

extend the profiling information with profile OPT summary

call graph and inline plan in runtime
-> stream to compilation cluster

### ThinLPO
https://www.researchgate.net/profile/Mehdi-Amini-4/publication/314105281_ThinLTO_Scalable_and_incremental_LTO/links/5f49899b458515a88b81c37b/ThinLTO-Scalable-and-incremental-LTO.pdf
`.o` contains size of function
-> think linking

## ICF
Identical Code Folding
same function body

## IPA-CP
cloning a function based on a context
constants, value ranges, devirtualization

## MOD / REF
## IPA-PTA
single compilation unit
## Devirtualization
## Visibility

