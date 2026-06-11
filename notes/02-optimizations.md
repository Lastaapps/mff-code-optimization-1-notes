# Scalar optimizations
- automatic variables of simple types
  - integers, floats, pointers, vectors
- with no address taken

## Control flow graph (CFG)
- basic blocks (BB)
  - sequences of instructions
  - edges are possible jumps

### Scopes
- local - inside a single BB
- global - across whole CFG
- inter-procedural - possibly whole program (minus libraries)

### Constant propagation
- locally -> forward propagation
  - držíme stav proměnných - pokud konstanta a je použita, dosadíme
  - ideálně chceme navázat constant foldingem

### Edge forwarding
- pokud zjistím, že je basic blok prázdný -> přeskočím
- spojené s odebráním nedostupného kódu

### Common subexpression elimination (CSE)
- stejné výrazy se počítají opakovaně -> spočtu jednou a uložím
- hash tabulka výrazů a odkazy na uložené výrazy pro invalidaci

### Copy propagation
- ukládání pospolností kopií/přiřazení
- `b=a; c=b;` => `c <- b <- a` => `c <- a`
- cooperates (repeats) with CSE

## Side effects
- calls const/pure
- volatile memory access
- control flow
- exceptions handling
- sometimes, memory access (nullptr dereference, ...)
- sometimes floating point

## Dead code removal
- removing statements without side-effects
- computing unused values
- backwards propagation
- reference counting in a hash table

## Super block optimization
- super block: group of basic blocks
- we take list of basic blocks as a single basic block and run it all again
- extended basic block: 
- hyperblock: 

# Scalar optimiztions
- constant prop
- common sub ellim
- copy propagation
- dead coe ellimintation
  - local: BB -> SB -> EBB
  - global
  - inter

## Solution to dataflow
- conservative
  - always correct
  - nic neodhadujeme, čekáme, co rozhodovací engine vymyslí
- maximal
  - pokud neznáme proměnnou, tipneme si ji a ověříme, zda premisa vyjde
  - už nejde zlepšit: S je maximální iff každé konzervativní řešení S' a každý BB IN_S(BB) >= IN_S'(BB) a OUT_S(BB) >= OUT_S'(BB)

- výchozí hodnoty "VAR"
- semi-latice - algebraická struktura s bin operací
  - x ^ x = x, x ^ y = z
  - nejvyšší prvek true, nejnižší false
  - separátní pro každou proměnnou
  - např.  0^true=0, 0^1=false
- jump functions - určují změnu stavu mezi začátkem a koncem BB
- ze začátku nenastavíme
- hledání chyb pomocí relaxce - iterative dataflow
- iterativní dataflow po skončení vydá konzervativní maximální řešení
- předpoklady:
  - monotonita: x<=x', y<=y' -> x^y<=x'^y'
- iterativní data-flow je konečný, pokud semi-latice má konečnou výšku

## Value range propagation
- držíme si interval (c_1, c_2) nebo anti-interval
- opět na semi-latice
  - (c_1, c_2) <= (c_1', c_2')
  - (c_1, c_2) ^ (c_1', c_2') = (min(c_1, c_1'), max(c_2, c_2'))
- např. if a>0 se nám rozpadne na 2 bloky
- relaxační algo nefunguje moc dobře - hluboká semi-laticsemi-e

- dataflow zpravidla malý
  - funce ~10 BB, každý ~5 instrukcí
  - ale z code generátorů / inlinování můžou vzniknout skutečně velké fce


## Liveness
- variable *a* is live at a given point *p* in program iff exists execution that defines *a* before *p* and uses it after *p* before def
- semi-latice s 2 stavy, top je 0, bottom je 1
- skip fce řeší, jestli je stále živá na začátku/konci
- backwards data flow
- BB_IN(a) is 1 iff BB uses *a* before def or BB_OUT(a)=1 and BB has no def of *a*

- dataflow is rapid if semi-latice is 0-1-latice and all entries are independent
