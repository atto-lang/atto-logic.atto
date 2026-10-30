# Pure Oreslang logic library: initial contract

## Implemented in this draft
- `src/propositional.ores`: implication, equivalence, exclusive-or, all/any/none reductions.
- `src/horn.ores`: positive propositional Horn rules with monotone least-fixed-point inference.
- `tests/logic_smoke.ores`: callable cross-backend smoke fixture (not yet executed).

## Semantics and bounds
- Closed-world *absence* is not classical negation. There is no negation-as-failure in v0.
- A fact is an opaque nonempty string atom; this initial version has **no logic variables**, substitution or unification.
- An empty-premise rule derives its head unconditionally.
- Duplicate facts collapse to one. Inference is deterministic for a given seed/rule sequence.
- `max_facts` causes an explicit error on exhaustion; the engine never reports a truncated closure as complete.
- Side-effect-free API except allocating a new closure. Caller-provided inputs must not be modified during evaluation.
- This is a reference implementation, not a claimed real-time, bounded-time or zero-allocation engine. It allocates arrays and linearly scans atoms.

## Future modules
1. `logic.term` and `logic.unify`: strongly typed immutable terms, occurs check, substitutions.
2. `logic.datalog`: typed relations, seminaive evaluation, indexed joins, stratified negation and safe rules.
3. `logic.relate`: fair search/miniKanren-style goals with explicit work budgets.
4. `logic.proof`: provenance witnesses and explanation DAGs.
5. `logic.induce`: tentative rule generation, scoring and train/holdout checks; *never* label induction as logical proof.
6. Constraints, incremental retraction and optional actor-owned indexes.

## Portability and required acceptance gates
Pure `.ores` source only. No native extension, JVM host binding, Java, C/C++, C#, JS, Python, foreign code generator, or other execution dependency.

Before merging: pin the exact commit SHA; parse/type-check all .ores files using genuine compilers; run `logic_smoke.run()` on Java, LLVM, WASM, Forge and .NET; compare return values and negative/budget tests; run independent review and required branch controls. A source-only or modeled check is *not* an executed cross-backend pass.
