# Intermediate languages
- AST specific to the given language

## Frontend
- lexical analysis (tokens)
- abstract syntax tree
- generation of IL (intermediate language) = IR (intermediate representation)

## Types
- struct, union, array, int, short, ...
- virtual tables and other metadata (GCC)

- High level
  - Generic, Gimple in GCC, LLVM
- Mid-level: atomic types, easy aggregates
- Low-level (RTL): CPU types
- Machine descriptors
  - QI, HI, SI, DI → quad, half, single - number of integer bits
  - HF, SF, DF, XF → float types
  - BI, PI - booleans, partial integers

## Symbol table
- symbol
  - variable or function
  - name (string)
  - visibility
    - public
    - local (static)
    - external
    - comdat - common data, merging symbols, inline
    - weak - optional symbols
- sections of the output binary:
  - empty/zero segment
  - data segment
  - robo data (symbol table?)
  - heap, stack

# Variables
- record in symbol table
- representation of constants - float, array, ...
  - big constants take a lot of memory when represented wrongly
- type and values

## Functions
- declaration, signature

### Function body
- control flow
  - high level:
    - if, while, ... and code blocks
    - OPEN MP processing
  - CFG - control flow graph
    - oriented multigraph
    - basic block - sequences of instructions executed in the given order 
    - vertices are basic blocks
    - edges are possible jumps
  - sequence of instructions
    - flattened CFG
- assignments
  - nested expressions, tuples
  - 2-address vs. 3-address code
  - need to handle flags as an outcome of arithmetic operations - different between platforms
  - storing instructions in lisp-like language
    - (INSN prev_ptr next_ptr (PARALLEL [(SET (REG.SI 0)(PLUS.SI (REG.SI 0) (REG.SI 1))) (SET (REG.CC 66))])
    - supports general optimizations
    - LLVM stores machine specific code, lower memory usage

# IL (Intermediate language)
- sybmol table (ECF)
types
- constants
- functions
  - control flow
  - arithmetics
  - memory (load, store, move)
  - function calls
  - exceptions handling
  - annotations
  - debug info (vs. optimizations)
