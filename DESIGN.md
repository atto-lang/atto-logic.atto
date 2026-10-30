# Oreslang Logic — userland library contract (provisional)

## Scope

All logic-engine source and tests are pure Oreslang (`.ores`), with no Java,
C++, C#, JavaScript, Python, foreign-runtime binding or external logic solver.
This is a library, not a new syntax dialect or required compiler extension.

## Source modules

- `propositional`: implication, equivalence, XOR and all/any/none reductions.
- `horn`: finite positive **propositional** Horn closure over string atoms.
- `term`: symbol/int/bool values, typed variables and structured application
  terms. Tags are checked at runtime; not yet compile-time generic relation types.
- `unify`: substitution-based first-order matching with an occurs check, type
  compatibility, bounded dereferencing depth and binding count.
- `datalog`: schema-checked relations with 1=symbol, 2=int, 3=bool columns,
  positive safe rules, semi-naive delta evaluation, deterministic first
  derivations, and explicit `max_facts`/`max_work` failures.
- `proof`: bounded dependency-ordered derivation trace for a present fact.
- `relate`: FIFO-fair scheduling of **finite equality-conjunction branches**.
  This does not implement general recursive miniKanren/stream suspension.
- `induce`: experimental finite feature-to-positive-label hypotheses using
  separate train and holdout precision. An accepted hypothesis is *not a proof*.

## Logic semantics

Datalog is positive and monotone. There is no negation-as-failure, stratified
negation, aggregates, equality join indexing, retraction or recursive search
streams yet. Rules must satisfy the range-restriction discipline: every head
variable appears in a positive body atom. Variables with the same rule-local
ID must agree on type throughout a rule. Empty-body rules are ground facts.

Positive Datalog least-fixed-point results are **set-valued**. Duplicates do
not increase the fact count. Each derived fact stores one witness, from a
previous stage, in a deterministic order. Provenance is not an enumeration of
every possible proof. A failed query is **not** a proof of its negation.

Typed terms distinguish symbol `"42"`, integer `42` and boolean `true`.
The Datalog evaluator accepts only flat scalar cells; structured application
terms are supported only in general unification. Unification failures return
`Match(matched=false)`; invalid/budget-limited work returns `Err`.

## Boundedness and memory caveats

- Datalog: `max_facts <= 10000`, `max_work <= 1000000`, schema/rule and
  arity limits. Exhaustion fails rather than returning an incomplete closure.
  Its work bound counts candidate join attempts, not actual CPU instructions.
- Propositional Horn: explicit fact-budget failure.
- Unification: explicit substitution depth and binding count caps. These are
  **not yet a total recursive-node-work bound** for wide compound terms.
- Relational search: finite branches only; result has `complete=false` when
  step/answer limits truncate evaluation.
- Induction: <=10000 rows with 1,000-point integer precision and explicit
  holdout support thresholds; input examples must be independently sampled
  and the caller must avoid subject/group leakage across train and holdout.
- Current `Array<T>` storage is appendable and not deeply frozen. Only call
  the engine on stable, actor-owned inputs. Nothing here establishes
  thread-safety, zero allocations, hard-real-time safety or lock freedom.

## Next engineering gates

1. Genuine compiler parse, typecheck and guest execution of `tests/main.ores`.
2. Expanded error classification and explicit total unification-work fuel.
3. Strongly typed relation APIs and actor-owned indexed/SoA data layouts.
4. Full proof DAG retention, alternative derivations and incremental retraction.
5. Stratified Datalog negation with verified strata; recursive relational
   streams with fair suspension and bounded cancellation.
6. Inductive rule generation with train/holdout/test, minimum description length
   or confidence intervals, and provenance distinct from deduction.

Cross-backend acceptance requires exact pinned compiler and package revisions,
real `.ores` execution and matched positive/negative results across
Java/GraalVM, LLVM AOT, WASM, Forge and .NET AOT. See
`docs/VERIFICATION.md`. No passing compiler results have been obtained.
