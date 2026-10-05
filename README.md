# Zen Browser on FreeBSD

Native FreeBSD build and compatibility notes for Zen Browser, with documented fixes validated on FreeBSD 14.5.

> Project status: successful native build and installation on FreeBSD 14.5-RELEASE-p1.
> Scope: reproducible build reference, FreeBSD compatibility fixes, and integration notes.
> This is not an official Zen Browser repository.

## Overview

This repository documents a native FreeBSD build of Zen Browser, including the compatibility correction required for Zen's icon resources on BSD platforms.

The goal is to preserve a reproducible record so the procedure can be repeated after future Zen, Firefox, LLVM, or FreeBSD updates.

The browser was built natively on FreeBSD. No Linuxulator, Linux binary, compatibility layer, Flatpak, or precompiled Linux package was used.

## Validated environment

| Component | Validated version |
|---|---|
| Operating system | FreeBSD 14.5-RELEASE-p1 amd64 |
| Desktop | KDE Plasma |
| CPU | Intel Core i7-9750H |
| RAM | 16 GB |
| Zen source | 1.23b |
| Firefox engine | 157.0 |
| Upstream sync commit | 813b28f44 |
| Surfer | 1.14.9 |
| LLVM | 19 |
| Compiler | clang / clang++ from LLVM 19 |
| Architecture | x86_64 / amd64 |
| Binary format | Native FreeBSD ELF |

Source was obtained from the official Zen repository with its submodules:

~~~sh
git clone --recurse-submodules https://github.com/zen-browser/desktop.git ~/zen-browser
~~~

The validated source state reports:

~~~text
Zen Browser 1.23b
Zen Twilight 1.24t
Firefox 157.0
Surfer 1.14.9
~~~

## FreeBSD compatibility fix

### Problem

Zen's icon resources were conditionally included only for Linux:

~~~cpp
#ifdef XP_LINUX
~~~

FreeBSD defines XP_FREEBSD, not XP_LINUX. The browser could therefore start successfully while producing errors such as:

~~~text
Missing chrome or resource URL:
chrome://browser/skin/zen-icons/...
~~~

This caused missing Zen interface icons.

### Fix

The resource condition in:

~~~text
browser/themes/shared/zen-icons/jar.inc.mn
~~~

was changed to include the BSD platform defines:

~~~cpp
#if defined(XP_LINUX) || defined(XP_FREEBSD) || defined(XP_NETBSD) || defined(XP_OPENBSD)
~~~

The exact patch is preserved in:

~~~text
patches/0001-freebsd-bsd-include-zen-icons.patch
~~~

### Result

After rebuilding, the expected Zen icon resources were present, including:

- algorithm.svg
- back.svg
- home.svg
- search-glass.svg
- circle-bars-filter.svg
- settings.svg
- trash.svg
- reload.svg

The browser UI was tested successfully with the icons functioning normally.

## Localization

The initial build exposed missing Library labels.

Zen's English localization source contains:

~~~text
locales/en-US/browser/browser/zen-library.ftl
~~~

The English localization data was synchronized into the generated engine tree using:

~~~sh
python3 scripts/update_en_US_packs.py
~~~

After rebuilding, the localization stage reported the missing entries as added/updated and the Zen Library labels displayed correctly.

This repository documents localization synchronization as a build step rather than presenting it as a source patch.

## Build configuration

The validated mozconfig was:

~~~text
ac_add_options --with-wasi-sysroot=/usr/local/share/wasi-sysroot
ac_add_options --with-libclang-path=/usr/local/llvm19/lib
ac_add_options --prefix=/usr/local
~~~

A copy is provided as:

~~~text
mozconfig.freebsd
~~~

Configure with LLVM 19:

~~~sh
CC=/usr/local/llvm19/bin/clang CXX=/usr/local/llvm19/bin/clang++ ./mach configure
~~~

Build:

~~~sh
CC=/usr/local/llvm19/bin/clang CXX=/usr/local/llvm19/bin/clang++ ./mach build -j2
~~~

The validated native executable was generated at:

~~~text
engine/obj-x86_64-unknown-freebsd14.5/dist/bin/zen
~~~

It was verified as a dynamically linked native FreeBSD ELF executable.

## FreeBSD Firefox patches

The FreeBSD Firefox port was used as a compatibility reference.

Firefox 157.0 from /usr/ports/www/firefox contained FreeBSD-specific patches. The available set of 37 FreeBSD Firefox patches was tested against the Zen/Firefox engine used for this build and validated successfully before being applied.

This is an important part of the methodology: FreeBSD-specific browser fixes should be evaluated against the exact Firefox/Zen engine version rather than blindly assuming patches from another Firefox release will apply.

## Installation

The native build installed successfully with:

~~~sh
sudo ./mach install
~~~

Installation completed successfully and produced:

~~~text
/usr/local/bin/zen
/usr/local/lib/zen/zen
~~~

The installed executable was verified as a native FreeBSD ELF binary.

## Packaging note

The build itself completed successfully.

The command:

~~~sh
./mach package
~~~

encountered a FreeBSD tar compatibility issue because the packaging logic expects the GNU tar option:

~~~text
--mode=go-w
~~~

This does not indicate a browser build failure. Native compilation and mach install completed successfully.

## KDE integration

mach install does not necessarily create a KDE application-menu entry for a manually installed FreeBSD build.

A desktop entry can be created under:

~~~text
~/.local/share/applications/zen.desktop
~~~

A generic example is provided in:

~~~text
desktop/zen-freebsd.desktop
~~~

After changing desktop metadata, KDE may require its application/icon cache to be refreshed.

## Build resource note

The complete browser build is resource-intensive.

The validated build used reduced parallelism:

~~~text
-j2
~~~

Additional temporary swap was used during compilation because of peak memory consumption. After the build, the temporary swap resources were removed and the system was returned to its original configuration.

The permanent swap configuration used by the validated system is a 16 GiB FreeBSD swap partition.

## Reproducibility

The key version anchors are:

~~~text
FreeBSD 14.5-RELEASE-p1
Zen 1.23b
Firefox 157.0
Surfer 1.14.9
LLVM 19
commit 813b28f44
~~~

Future builds should record the corresponding Zen commit and Firefox engine version because Zen is actively synchronized with upstream Firefox.

## Repository contents

~~~text
.
├── README.md
├── BUILD.md
├── mozconfig.freebsd
├── desktop/
│   └── zen-freebsd.desktop
└── patches/
    └── 0001-freebsd-bsd-include-zen-icons.patch
~~~

This repository deliberately does not contain the complete generated Zen build tree or build artifacts. The upstream Zen source remains the canonical source; this repository preserves the FreeBSD-specific knowledge and corrections needed to reproduce the validated build.

## Upstream

Zen Browser source: https://github.com/zen-browser/desktop

FreeBSD: https://www.freebsd.org/

## Disclaimer

This project is an independent FreeBSD build/reference project. It is not affiliated with or endorsed by the Zen Browser project.

Zen Browser source, Firefox components, and other upstream materials remain subject to their respective licenses and copyrights.
