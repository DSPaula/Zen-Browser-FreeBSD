# Source Recovery

This repository preserves the native FreeBSD build work as a reproducible source delta.

## Engine base

The modified Firefox engine was based on:

- Repository: `zen-browser/desktop` engine subrepository
- Base revision before Zen/FreeBSD engine changes: `d3ec66377c`

The working tree produced the successful native FreeBSD build after the Zen patch set and the FreeBSD Firefox patches were applied.

## Functional engine delta

A local recovery archive was created and verified from the working tree:

- Archive: `zen-engine-functional-delta.tar.gz`
- Size: 13,193,516 bytes
- Tracked-file binary patch: 4,326,658 bytes
- Additional untracked files: 181
- `.bak` backup files excluded
- Build artifacts excluded

The archive contains:

1. `engine.patch` — the exact binary-capable Git diff from `d3ec66377c`
2. `untracked/` — the 181 additional source/resource files required by the transformed engine tree

The names and paths of all 181 untracked files were independently compared against the working tree and matched exactly.

## Recovery procedure

Start from a clean checkout of the engine at revision `d3ec66377c`.

From the engine root:

```sh
git apply --binary engine.patch
```

Then restore the files contained in `untracked/` into the engine root, preserving their relative paths.

Do not restore `.bak` files.

## Important distinction

This is a **functional delta against the exact base engine revision**, not a standalone copy of the complete Firefox/Zen source tree. The upstream base source must therefore be obtained at `d3ec66377c` before applying this recovery delta.

The successful build was performed on FreeBSD 14.5-RELEASE-p1 amd64 with LLVM 19.
