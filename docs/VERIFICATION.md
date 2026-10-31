# Genuine compiler-backed execution gate

**State: BLOCKED / NOT EXECUTED (2026-10-30).** Source-only audits are not
compiler parsing, type checking, guest execution or backend parity.

## Scope: only Oreslang guest code

All implementation and regression programs are `.ores`. A compiler, CI
runner, Maven/Clang build system or shell harness may be used to **execute**
these programs, but none may implement or replace logic operations in the
library. No Java/C++/C#/JavaScript/Python/FFI solver is allowed in this package.

There are three independent admission levels; do not collapse them.

### Gate 1: runner and genuine compiler canary

For each backend (Java/GraalVM, LLVM, WASM, Forge, .NET):

1. Record exact package-tree SHA, compiler source SHA, compiler binary SHA-256,
   runtime variant, execution host and runner ID; require independent provenance.
2. Require real runner assignment: a completed run with **at least one executed
   step**, an actual compiler process and actual guest process.
3. Compile `tests/compiler_canary.ores` using that backend's authentic
   compiler. Execute the compiled guest program. Require exit status **0**.
4. Do not replace the guest with a host-side model or compare only text/IR.

The canary deliberately exercises scalar syntax only. Passing it proves
compiler + runner admission, **not** the logic-engine behavior.

### Gate 2: full-library compilation and execution

1. On the same pinned revisions, compiler-parse and type-check
   `tests/main.ores` and its *entire import closure*, including
   `src/term.ores`, `src/unify.ores`, `src/datalog.ores`,
   `src/proof.ores`, `src/relate.ores`, `src/induce.ores`,
   `src/horn.ores`, `src/propositional.ores`, and all four
   `tests/logic_*.ores` suites.
2. Execute the compiled `tests/main.ores` guest and require exit status **0**.
   Exit codes 1, 2, 3, 4 respectively identify baseline, advanced,
   edge-case and Horn/Datalog differential fixture failure; all nonzero
   statuses must fail the gate.
3. Check negative assertions: occurs-check and sort mismatches,
   unsafe head variables, wrong schema, duplicate schema, invalid budgets,
   incomplete search flags, missing proof and held-out hypothesis rejection.
   Assert `unify_bounded` rejects work-budget exhaustion for wide terms,
   late occurs checks, and alias chains; assert Datalog propagates one
   budget across join candidates and unification.
4. No skipped tests, unexecuted compiler invocations, silent fallback to a
   host-only emulator, or 'model pass' counted as an execution pass.

The Java/GraalVM `dev.oreslang.launcher.OresMain` has a real `--check`
option; use it for compile-only checking, then invoke the actual guest
execution CLI (without `--check`). The Java compiler must be built from a
pinned commit using its own documented build process, not fabricated inside
this package. The native LLVM README documents the genuine
`oreslang-llvmc` -> LLVM IR -> `clang` AOT execution path. Run the canary
with the native compiler first. Its current documented general heap/object
limitations block a full-library LLVM pass; keep this as a visible parity
blocker until corrected and executed. Do not assert the other backends pass
without corresponding exact-head evidence.

### Gate 3: behavioral equivalence and safety

- Require identical answers, error classes, proof witness ordering and resource
  exhaustion across all five backends.
- Run repeated, randomized **Oreslang** fixture generators, adversarial
  depth/width budgets, cancellation, actor generation release, sanitizers
  and memory ownership checks.
- Benchmark allocation count, bytes per fact, index memory, and worst-case tail
  latency before claiming zero-allocation or hard-real-time suitability.
- Obtain independent code review, enforced non-empty required checks and branch
  protection before merging.

## Source-org evidence at inspection

On 2026-10-30 the logic repository had no GitHub Actions runs. The Java and
LLVM source organizations' latest sampled jobs failed with **zero executed
steps** (Java run `41110063580`, job `124249866636`; LLVM run
`41110095477`, job `124249977129`). Neither Java nor LLVM had a
published GitHub Release containing a ready-to-use compiler executable. The
development container had `java` but **no** `oreslang` or
`oreslang-llvmc` executable.

Do not mirror any private source or PR tree into an alternate organization
solely to escape Actions quota. First verify governance/admission and an
approved destination allowlist, with exact SHA and per-blob integrity,
independent reviews, and source-owner approval. Prefer authenticated
same-repo/private runners that cannot expose the code to outside actors.

## Next action

Provision or obtain **one** genuine pinned Oreslang compiler executable,
execute `tests/compiler_canary.ores`, then attempt the full library on
that same compiler. Record the **first actual parser/type/ownership diagnostic**
rather than guessing which source construct fails. Fix in the logic PR if
the source is invalid; if a valid guest construct is missing in the backend,
open a backend-specific parity issue. Continue other backends only after
recording real runner and executable evidence.

## October 30: latest runner-admission diagnosis

The same-organization Java/GraalVM integration workflow remained **NOT
EXECUTED** at [run #41157594708](https://github.com/ores-truffle-oreslang/oreslang-source.java/actions/runs/41157594708),
commit `61badbd0bb8b4dcf12d0c7b798b6ebb03f10ad2a`.
The Actions job API reported `runner_id=0`, an empty `runner_name`,
**zero job steps**, and termination roughly three seconds after job start.
The corresponding check-run metadata reports **two annotations**, but their
contents are inaccessible through the current read-only repository connector.
Do not guess the message or classify this as an Oreslang test failure.

An independent source-free admission probe in private
`honeypot-r-us-test/hnpt-docs`, [run #41153246476](https://github.com/honeypot-r-us-test/hnpt-docs/actions/runs/41153246476),
also terminated without any job steps. Earlier success on that carrier does
not prove *current* runner availability, and no private Oreslang code was
copied there.

**Admin investigation, without another redundant workflow run:**

1. Read the exact failed-check annotations in GitHub's web UI and record
   their text without credentials, email notification tokens, or secrets.
2. If they indicate billing, inspect the owning organization's
   **Settings → Billing & Licensing → Payment information** for past-due
   charges and the GitHub Actions budget, metered usage, and
   stop-usage flag. Budget room alone does not prove payment eligibility.
   See GitHub's [troubleshooting guide](https://docs.github.com/en/actions/how-tos/troubleshoot-workflows)
   and [billing guide](https://docs.github.com/en/billing/how-tos/set-up-payment/manage-payment-info).
3. Check repository and organizational **Settings → Actions → General**
   for restrictions on GitHub-hosted runners and required actions. A
   GitHub-controlled Actions suspension may require GitHub Support.
4. If the UI shows neither billing nor policy blockage, ask GitHub Support
   to investigate the recorded run/job/check identifiers, including whether
   runner provisioning is blocked by the account-level execution gate.
5. Only after resolving the documented cause, run a source-free one-step
   admission canary. Demand a nonzero `runner_id`, actual executed steps,
   and a successful completed process before retrying the full compiler.
6. Refresh the pinned `ci/logic_lib/` snapshot from current PR #2,
   verify all Git blob SHAs against that exact PR head, then execute the
   genuine GraalVM compiler canary and four logic suites. The existing
   snapshot is **stale whenever PR #2 receives source changes**.

Never infer a compiler pass from static file checks, a queued run, or an
Actions job that was never assigned a runner. Do not transfer private source
across organizations merely because an older carrier run once succeeded.
