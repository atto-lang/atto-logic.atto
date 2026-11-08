# atto-logic.atto

Logic programming and declarative reasoning experiments using AttoLang.

## Scope

Rules, logical inference, and composable declarative control flow.

## API and portability principles

- Specify numerical/semantic behavior before assuming support on every backend.
- Document input types, error behavior, shape and memory ownership requirements as applicable.
- Distinguish CPU execution, SIMD, GPU offload, and other target-specific capabilities.

## Verification

Use checked-in tests, documented tolerances, and exact-commit runtime evidence. Host-only or compilation tests are not equivalent to executed AttoLang guest tests. Record backend gaps explicitly.

## Development status

This describes the repository's intended purpose, not a verified list of implemented features. Refer to checked-in code and CI for current functionality and validated commands.

## Related

- [AttoLang](https://github.com/atto-lang)
- [ML conformance](https://github.com/atto-lang/atto-ml-conformance)
- [Language specification](https://github.com/atto-lang/atto-spec)

## Repository ownership and contribution routing

**Belongs here:** Logic-programming experiments in native AttoLang, rules, inference, constraint evaluation and composable declarative APIs.

**Belongs elsewhere:** General language semantics in atto-spec; graph modeling in atto-stategraph.atto; example usage in atto-demos.

**Security and portability boundary:** Bound search depth and memory, avoid dynamic reflection that breaks AOT, and reject nontermination/unsupported inference modes explicitly.

AttoLang is the public name; preserve existing Oreslang/.ores compatibility and use canonical .atto for new portable library source. This README defines scope, not feature readiness.

**Acceptance:** name exact source SHA/tree, runtime/toolchain, positive and negative checks and execution receipt before claiming support. Host-only tests and empty CI steps do not prove guest execution.
