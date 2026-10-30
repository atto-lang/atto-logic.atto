# Verification gate (do not claim executable parity without runs)

This package contains only Oreslang implementation files. Test fixtures under
`tests/` are also pure Oreslang. The package is a userland library, not a
compiler dialect and not linked to Java/C++/.NET/JS/Python/FFI.

## Required execution proof

1. Pin the package branch commit and each genuine compiler/runtime source SHA.
2. Build a genuine Oreslang compiler and record executable SHA-256. Do not
   substitute a host implementation or an isolated source-free model.
3. Parse/type-check **every** `.ores` file; require no skipped or unrecognized
   constructs; verify import/manifest resolution and ownership diagnostics.
4. Compile and execute `tests/main.ores` in Java/GraalVM, LLVM AOT, WASM,
   Forge and .NET AOT, checking exit status 0.
5. Run negative tests in `tests/logic_advanced_smoke.ores`: occurs check,
   type mismatch, unsafe range-restriction, proof budget, work budget, fact
   budget, and relational-search truncation.
6. Add independent differential oracle tests, adversarial depth/size tests,
   cancellation tests, sanitized native runs, and hot-reload actor-lifetime
   checks before production usage or resource guarantee claims.
7. Require source-review and branch-protection gates. No private source mirror
   to a carrier until repository security/admission policy is independently
   verified and approved.

## Status

Source committed; compiler-backed parsing, type checking, running and
five-backend parity **NOT EXECUTED/NOT VERIFIED** as of 2026-10-30.
No compiler executable is available in the current local environment.
Do not treat source-only static checks as executable tests.

## Portability hazards to verify

- Recursive `Term` arrays and class ownership/borrowed return paths.
- `Array<T>` constructors, `add`/`get`/`length()` and mutable locals.
- Import graph and package namespace rules.
- `Result` error propagation, constructor visibility and method declaration forms.
- Strict budgets and error results on every backend.
- No guarantee of zero allocations, hard real time, frozen arrays, or
  deep immutability of values stored in publicly accessible arrays.
