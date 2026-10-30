# Oreslang Logic

Pure **userland Oreslang** logic programming: deductions, reductions,
typed relational rules, unification, proof tracing, fair finite searches and
experimental inductive hypothesis scoring.

**No Java, C++, C#, JavaScript, Python, FFI or external logic engine is used
to implement the library.** All implementation files are `.ores`, and the
regression suites are `.ores` too. No special compiler dialect is required
by the design, although current backend compilers may not yet support every
language construct used here.

## Modules

| Source | Purpose |
| --- | --- |
| `src/propositional.ores` | implication, equivalence, XOR, all/any/none |
| `src/horn.ores` | bounded propositional positive Horn deduction |
| `src/term.ores` | typed variables, values, structured terms |
| `src/unify.ores` | first-order unification with occurs check |
| `src/datalog.ores` | schema-typed positive Datalog with semi-naive joins |
| `src/proof.ores` | first-witness provenance extraction |
| `src/relate.ores` | FIFO-fair finite equality-goal branches |
| `src/induce.ores` | train/holdout feature implication scoring |

## Example: deductions

Use schemas `parent(symbol, symbol)` and `ancestor(symbol, symbol)`.
Add seed facts `parent(alice,bob)` and `parent(bob,carol)`.
Rules:

```text
ancestor(X,Y) :- parent(X,Y).
ancestor(X,Z) :- parent(X,Y), ancestor(Y,Z).
```

Calling `datalog.evaluate(schemas, facts, rules, max_facts, max_work)`
computes the least fixed point, including `ancestor(alice,carol)`.
Use `proof.explain(knowledge, target, max_nodes, max_depth)` to obtain
one ordered derivation. This is a description of the fixture in
`tests/logic_advanced_smoke.ores`, **not a claim that it has run**.

Datalog column types are schema-checked at runtime: symbol, int, bool.
Unknown relations, ill-typed values, unbound head variables and exhausted
resource budgets return `Err`; there is no silent truncation of deductions.

## Tests and status

- `tests/logic_smoke.ores`: propositional and baseline Horn tests.
- `tests/logic_advanced_smoke.ores`: recursive rules, proof trace, typed
  unification, occurs check, budgets, unsafe rules, fair search and induction.
- `tests/logic_edge_cases.ores`: indirect cyclic terms, invalid schemas,
  empty-body rules, zero-arity facts, invalid limits and holdout rejection.
- `tests/compiler_canary.ores`: independent scalar compiler/runner admission
  test; it does **not** exercise the logic engine.
- `tests/main.ores`: runs all three logic suites; exit `0` means success,
  `1`/`2`/`3` identify the failing suite.
- `.ores.pkg.toml`: package discovery and source roots.

**Execution status: not yet verified.** No genuine compiler has run these
fixtures in this development session. The library does not claim five-backend
parity or zero allocations. See [DESIGN.md](DESIGN.md) and
[verification gates](docs/VERIFICATION.md).

## Future optimization

Prefer actor-owned persistent relations, indexed joins, and SoA/AoSoA
representations before using this evaluator in latency-sensitive loops.
These are implementation improvements, not changes to the library's
logical semantics.
