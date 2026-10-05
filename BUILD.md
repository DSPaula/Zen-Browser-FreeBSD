# Native Zen Browser Build on FreeBSD 14.5

This document records the build procedure successfully validated on FreeBSD 14.5-RELEASE-p1 amd64.

## 1. Obtain the source

git clone --recurse-submodules https://github.com/zen-browser/desktop.git ~/zen-browser
cd ~/zen-browser

Confirm the source revision:

git describe --tags --always --dirty

Validated result: 1.23b

Upstream synchronization commit:

813b28f44 gh-15565: Sync upstream Firefox to version 157.0 (gh-15566)

## 2. JavaScript dependencies

npm ci

## 3. Download and import build dependencies

npm run download
npm run import

The validated import completed all 264 Zen git patches.

## 4. Synchronize English localization

python3 scripts/update_en_US_packs.py

The generated tree should contain:

engine/browser/locales/en-US/browser/zen-library.ftl

This step fixed the initially missing Zen Library labels.

## 5. Apply the FreeBSD icon-resource correction

The relevant source file is:

browser/themes/shared/zen-icons/jar.inc.mn

The FreeBSD-compatible condition is:

#if defined(XP_LINUX) || defined(XP_FREEBSD) || defined(XP_NETBSD) || defined(XP_OPENBSD)

The exact change is preserved in:

patches/0001-freebsd-bsd-include-zen-icons.patch

## 6. Configure

Use mozconfig.freebsd.

CC=/usr/local/llvm19/bin/clang \
CXX=/usr/local/llvm19/bin/clang++ \
./mach configure

## 7. Build

The validated build used two parallel jobs:

CC=/usr/local/llvm19/bin/clang \
CXX=/usr/local/llvm19/bin/clang++ \
./mach build -j2

Validated output:

engine/obj-x86_64-unknown-freebsd14.5/dist/bin/zen

Verify:

file engine/obj-x86_64-unknown-freebsd14.5/dist/bin/zen

Expected result: native FreeBSD amd64 ELF executable.

## 8. Install

sudo ./mach install

Validated installation:

/usr/local/bin/zen
/usr/local/lib/zen/zen

## 9. Verify

file /usr/local/lib/zen/zen
/usr/local/bin/zen

Zen should launch natively on FreeBSD without Linuxulator.

## 10. Packaging caveat

./mach package

may fail on stock FreeBSD because Mozilla's packaging logic expects GNU tar's:

--mode=go-w

This is independent of the successful native compilation and installation.

## 11. Verification checklist

- [ ] Binary identified as native FreeBSD ELF.
- [ ] Zen starts without Linuxulator.
- [ ] Zen Library labels are present.
- [ ] Zen interface icons are present.
- [ ] /usr/local/bin/zen launches the installed browser.
- [ ] KDE integration works after adding a desktop entry if desired.

## 12. Reproducibility principle

Always record:

1. FreeBSD release and patch level.
2. Zen tag/commit.
3. Firefox engine version.
4. LLVM version.
5. FreeBSD-specific patches.
6. Localization synchronization state.
7. Packaging/toolchain incompatibilities.

This makes future failures distinguishable between source regressions, FreeBSD port changes, LLVM changes, and packaging-tool differences.
