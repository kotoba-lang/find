# kotoba-lang/find — org-wide recursive search in Kotoba

Search a directory tree for `.clj` / `.cljc` / `.cljs` files by extension and
by content (grep), as a Kotoba `.kotoba` guest that runs on the kbb host with
the `:fs/browse-dir` capability (id 261).

The kotoba-guest replaces the slow per-file nbb scripts
(`scripts/repo-search.cljs`, `scripts/langchain-store-adoption-scan.cljs`,
`scripts/jvm-dependency-scan.cljs`) with a single recursive walk over the
granted directory TREE, every descent inside the capability guard.

## How it works

`fs/browse-dir` (capability id 261, kotoba-core-contracts 2ff3f736) answers
ONE directory as a `"\n"`-joined listing where each line is
`"<name>\t<0|1>"` (0 = file, 1 = directory) — so the guest can decide whether
to descend, and walk the whole tree itself. Same scope narrowing as
`:fs/browse` (granted directory tree).

The guest (find/core.kotoba) walks a granted root:

- lists the root with `browse/entries-dir`
- splits each line on the last byte (the flag) and the `\t`
- descends into entries whose flag is `1`, reads files whose flag is `0`
- filters by extension and/or content (string-index-of)

Every `fs-browse-dir` and `fs/app-data` call is inside the capability guard,
so a walk cannot escape the granted tree.

## Capabilities

Policy grants `:fs/browse-dir` (the tree to walk) and `:fs/app-data` (to read
file contents) with explicit resource scopes — fail closed if either is
granted without a directory scope.

## Finding the lib

- `browse/entries-dir` in `kbb.browse` (kotoba repo, Slice C)
- `fs/browse-dir` capability id 261 (kotoba-core-contracts)
- `:fs/browse-dir` catalogue entry / `:host/fs-browse-dir` effect row
  (kotoba-lang + kotoba-sema catalogues, amu pin 8e5a1af2)

## Build

Requires amu pin >= 8e5a1af2 (carries the fs/browse-dir capability in its
kotoba-sema catalogue). Check:

```
amu check <abs-path>/find/core.kotoba --jvm-free
```