# STATUS — fs/browse-dir capability chain (2026-09-07, closing)

Goal: an org-wide recursive search lib (`find`/`grep`) in Kotoba that walks a
directory TREE (not one dir), so the slow per-file nbb scripts
(`scripts/repo-search.cljs`, `scripts/langchain-store-adoption-scan.cljs`,
`scripts/jvm-dependency-scan.cljs`) can be replaced by a single `.kotoba`
guest with a granted-tree capability.

Why the extra capability: the existing `:fs/browse` answers entry NAMES only
with no way to ask "is this entry a directory" — a recursive tree walk needs
that flag. `:fs/browse-dir` answers one dir as "<name>\t<0|1>" lines.

## Landed on main (verified by GitHub compare, ancestor of main)

| repo | change | ref |
|---|---|---|
| kotoba-core-contracts | `:fs/browse-dir` capability id 261 | 2ff3f736 |
| kotoba-lang | `:host/fs-browse-dir` effect row + fs-browse-dir grammar head | cb41cb62 |
| kotoba-lang | `:fs/browse-dir` in capability-catalogue (both copies) | 85fcca93 |
| grammar | vendored grammar resync (fs-browse-dir head) | 7fc25abf |
| kotoba-sema | grammar resync + capability catalogue vendor | af8cc780, 8e5a1af |
| kotoba | grammar resync + pins | cb70c446 |

## Open / unverified

| item | state |
|---|---|
| **kotoba runtime provider** | PR kotoba#598 (branch agent/fs-browse-dir-provider, 7741c8cbf). Provides `fs-browse-dir` in host_providers.clj + kbb.clj + runtime.clj + kbb_js.cljs + lib/kbb/browse.kotoba (`entries-dir`). NOT merged. |
| **amu compile lower** | AMU PR #866 (sema 8e5a1af2) CLOSED 2026-07-07. The sema pin advance is coupled to murakumo's KIR drift gate and breaks it. |
| **find/core.kotoba** | This repo's initial `.kotoba` walker, UNVERIFIED (needs amu/compile with fs/browse-dir admitted). |

## Blocking / inconsistency (must resolve before find is complete)

- **Compiler wire-id 36**: sema main `93a2ad4` (PR #69, "resync :fs/browse-dir compiler wire 36") re-registered `:fs/browse-dir` compiler-wire-id as **36** (amu KEXE import slot), while the runtime capability id stays **261** and MY kotoba runtime wire map uses **261**. The kbb_js / compile lowering must align to the catalog number **36** before the lib can compile. `kbb_js.cljs` wire-ids map and the find guest must be checked/moved to 36.
- **murakumo KIR-drift**: any amu sema pin advance (to a sema commit with min/max heads ccd3b23 or later) fails `murakumo.kotoba-oracle-authority-test` ("kotoba oracle not ready"). Resolving requires a murakumo-coupled wave or pinning amu's sema to a commit WITHOUT the KIR-changing heads — none exists on sema main today.

## Next concrete step (resume here)

1. Reconcile the wire id to **36** in `kbb_js.cljs` + find lib (the K8S-write map says 261; the catalogue says 36).
2. Land kotoba PR #398 (provider) once wire id is consistent.
3. Verify find/core.kotoba with `amu check --jvm-free` once the sema/catalogue chain admits `:fs/browse-dir`.
4. If amu-compile lower is required, do a murakumo-coupled wave (murakumo oracle KIR regenerate + amu sema pin together), not an isolated amu bump.

Handoff owner: itonami (hermes profile). ADR trail: superproject `kotoba-refactor-plan.md`.