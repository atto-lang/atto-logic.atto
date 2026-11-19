> **AI agent instructions (organization-wide):** Before working on code, issues, or pull requests, read and follow the [organization `AGENTS.md`](https://github.com/ores-truffle-oreslang/.github/blob/main/AGENTS.md), including its **merge-first stealth-development policy**. Honor any repository-local `AGENTS.md`, explicit holds, and GitHub-enforced rules.

# Oreslang Logic

## Oreslang to AttoLang migration (2026-11-18)

**AttoLang** (formerly **Oreslang**) is the current public language identity. This `ores-truffle-oreslang/oreslang-logic.ores` checkout retains old naming. Its AttoLang counterpart for new contributions is [atto-lang/atto-logic.atto](https://github.com/atto-lang/atto-logic.atto); this link **does not imply an in-place transfer, identical history, migrated issues/PRs or feature parity**. The old source and commands below remain useful as historical checkout documentation, not a guarantee of current language conformance.

**[atto-lang/atto-spec](https://github.com/atto-lang/atto-spec)** is the canonical language specification. The portable library source belongs in [atto-common-source](https://github.com/atto-lang/atto-common-source), shared positive/negative guest tests in [atto-platform-conformance](https://github.com/atto-lang/atto-platform-conformance), and compiler-specific implementations in `atto-source.*`. The language's current proposed direction includes canonical `.atto`, `snake_case`, `borrow!`/`take!`/`share!`/`copy!`/`copy!!`, [implicit borrow defaults](https://github.com/atto-lang/atto-spec/blob/main/language/implicit-argument-borrow-v1.md), [`define const` including permitted `os.read_env`](https://github.com/atto-lang/atto-spec/blob/main/language/file-module-const-v1.md), and [`<<<`/`>>>`/`ashr` shifts vs. `<<`/`>>` streams](https://github.com/atto-lang/atto-spec/blob/main/language/bitwise-and-stream-operators-v1.md). Check individual spec admission status; **a design target is not proof of execution**.

Keep legacy `.ores` fixtures, CLI/package/ABI names, and version-pinned commands where they document code still in this tree. Avoid Python in new AttoLang tooling/tests/CI; Rust support tools are preferred. Only claim cross-backend support with exact SHA/tree, target/toolchain, real executed positive and negative guest tests; report failed, unsupported and not-executed explicitly. Empty-step/host-only CI is not guest conformance.

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
| `src/unify.ores` | occurs-checked unification with shared operation fuel |
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
- `tests/logic_edge_cases.ores`: indirect cyclic terms, wide-term
  fuel exhaustion, alias chains, Datalog shared work limits, invalid schemas,
  empty-body rules, zero-arity facts and holdout rejection.
- `tests/compiler_canary.ores`: independent scalar compiler/runner admission
  test; it does **not** exercise the logic engine.
- `tests/logic_differential.ores`: independent positive Horn/Datalog
  fixed-point equivalence checks, recursive cycles and fact-budget failures.
- `tests/main.ores`: runs all four logic suites; exit `0` means success,
  `1`/`2`/`3`/`4` identify the failing suite.
- `.ores.pkg.toml`: package discovery and source roots.

A new `.github/workflows/source-integrity.yml` checks exact checkout identity,
required source/guest-test file presence, and three negative structural controls
(missing, empty, symlink). **That workflow is source-only, not compiler or guest
execution**; a green status does not certify logic semantics or native parity.

**Execution status: not yet verified.** No genuine compiler has run these
fixtures in this development session. The library does not claim five-backend
parity or zero allocations. See [DESIGN.md](DESIGN.md) and
[verification gates](docs/VERIFICATION.md).

## Future optimization

Prefer actor-owned persistent relations, indexed joins, and SoA/AoSoA
representations before using this evaluator in latency-sensitive loops.
These are implementation improvements, not changes to the library's
logical semantics.
