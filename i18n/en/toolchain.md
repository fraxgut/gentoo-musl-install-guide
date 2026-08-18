<!--
i18n/en/toolchain.md
@fraxgut
CC-BY-SA-4.0
Toolchain: the LLVM/Clang profile, hardening and link-time optimisation
-->

# LLVM toolchain and LTO

> 🌐 **Language:** [Latina](../la/instrumenta.md) · [Español](../es/herramientas.md) · **English**

This document shows how the installation combines musl, the hardening of the
`hardened` profile and the LLVM/Clang toolchain. It also shows how to enable
link-time optimisation.

## Contents

1. [The profile problem](#the-profile-problem)
2. [What each profile gives](#what-each-profile-gives)
3. [The stacked profile](#the-stacked-profile)
4. [Known friction](#known-friction)
5. [The make.conf configuration](#the-makeconf-configuration)
6. [Link-time optimisation](#link-time-optimisation)
7. [Rebuild the system](#rebuild-the-system)
8. [The kernel with Clang](#the-kernel-with-clang)

## The profile problem

Gentoo has separate profiles for musl with hardening and for musl with LLVM:

```
default/linux/amd64/23.0/musl/hardened   (exp)   →  GCC
default/linux/amd64/23.0/musl/llvm       (exp)   →  Clang, LLD, libc++
```

No official profile combines them. There is also no `musl-llvm-hardened`
stage3. The C library and the toolchain become fixed when you unpack the
stage3. You thus make the decision before the installation, and you cannot
change it later without a rebuild of the system.

This guide starts from the `musl-llvm-openrc` stage3. It selects the
`musl/llvm` profile. Then it **puts the features of the `hardened` profile on
top** through a profile of its own. Portage permits this composition: a profile
can declare several parents, and it inherits from all of them.

This method has a precedent. In the Gentoo forums, a user installed from the
musl-clang tarball and then added the hardening with a custom profile. The wiki
also documents the same syntax to build LLVM desktop profiles.

## What each profile gives

The difference is smaller than the names suggest. You must know what you get
and what stays open.

The 23.0 profiles enable these features by default, **in all cases**:

- Position independent executables (PIE)
- Stack protection with `-fstack-protector-strong`
- Stack clash protection
- RELRO and `BIND_NOW`

The `hardened` profile adds these items:

| Item                                       | Source                                    |
|--------------------------------------------|-------------------------------------------|
| `USE="hardened pic xtpax"`                  | `features/hardened/make.defaults`         |
| `USE="cet"` on amd64                        | `features/hardened/amd64/make.defaults`   |
| `PROFILE_IS_HARDENED=1`                     | `features/hardened/make.defaults`         |
| `pie` in `use.force`                        | `features/hardened/use.force`             |
| `sys-apps/elfix` in the base set            | `features/hardened/packages`              |
| `xattr` forced on tar, coreutils and portage | `features/hardened/package.use.force`    |
| `-D_FORTIFY_SOURCE=3` in place of 2         | The toolchain of the profile              |
| Standard library assertions                 | The toolchain of the profile              |

The stack does not give you the last two rows, because they come from the GCC
configuration and not from a profile variable. You set both by hand in
`make.conf`. The section below shows how.

One honest note about `xtpax`: the flag marks binaries with extended PaX
attributes, and the same profile sets `PAX_MARKINGS="none"`. Without a kernel
with PaX — and no public kernel has it now — the marks change nothing at run
time. This guide keeps the flag because it belongs to the set that defines the
profile, and because it costs nothing. It is not an active defence.

## The stacked profile

Portage needs the profile in a repository. Make a local one:

```bash
emerge app-eselect/eselect-repository dev-vcs/git
eselect repository create local
```

Check that `metadata/layout.conf` declares the profile format that permits the
`repository:path` syntax. `eselect repository create` does not always write it:

```bash
cat /var/db/repos/local/metadata/layout.conf
```

```ini
masters = gentoo
profile-formats = portage-2
```

Make the profile and declare its parents:

```bash
mkdir -p /var/db/repos/local/profiles/musl-llvm-hardened
echo 8 > /var/db/repos/local/profiles/musl-llvm-hardened/eapi

cat > /var/db/repos/local/profiles/musl-llvm-hardened/parent <<'EOF'
gentoo:default/linux/amd64/23.0/musl/llvm
gentoo:features/hardened
EOF
```

The sequence is important. Portage reads the parents in order, and it
accumulates the incremental variables such as `USE`. Thus `features/hardened`
adds its flags and keeps the flags of the LLVM profile. The variables that are
not incremental stay unchanged. These are `CC`, `CXX`, `LD` and the other tools
from `features/llvm`, because the `hardened` profile does not set them.

Register the profile, thus `eselect` shows it:

```bash
echo "$(portageq envvar ARCH) musl-llvm-hardened exp" \
    >> /var/db/repos/local/profiles/profiles.desc
```

Select it:

```bash
eselect profile list
eselect profile set <number>
eselect profile show
```

Check that the stack works:

```bash
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

The first command must show the four flags. The second command must print
`clang`.

## Known friction

`features/hardened/make.defaults` sets `USE="... -jit -orc"`. On a system with
GCC those flags do almost nothing. Here they disable the LLVM run-time compiler
and its ORC layer, and some packages need them. Mesa is the usual example: the
software renderer and some OpenCL paths use the LLVM JIT.

If a package that you need depends on those capabilities, enable them for that
package. Do not remove them from the profile:

```bash
mkdir -p /etc/portage/package.use
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

The other packages keep `-jit -orc`.

## The make.conf configuration

```ini
# /etc/portage/make.conf

# --- Toolchain ---
# The musl/llvm profile sets CC, CXX, LD and the LLVM tools. To declare
# them again here only repeats what the profile guarantees.

# --- Hardening ---
# The 23.0 profile gives PIE, SSP, stack clash protection, RELRO and
# BIND_NOW. These two macros give what the hardened profile gets from the
# GCC configuration, and what Clang does not inherit: level 3 of
# FORTIFY_SOURCE, and the assertions of the C++ standard library.
HARDENING_FLAGS="-D_FORTIFY_SOURCE=3"
HARDENING_CXXFLAGS="-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST"

# --- Optimisation ---
# ThinLTO is the LTO variant that Gentoo recommends for Clang. It links in
# parallel, and it uses much less memory than monolithic LTO.
# The warnings that become errors show incompatibilities between
# translation units. Those problems appear only at link time.
WARNING_FLAGS="-Werror=odr -Werror=strict-aliasing"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${HARDENING_FLAGS} ${WARNING_FLAGS}"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS} ${HARDENING_CXXFLAGS}"
LDFLAGS="${COMMON_FLAGS} ${LDFLAGS}"

# --- Parallel jobs ---
# Replace the number with the output of 'nproc'. make.conf does not expand
# commands.
MAKEOPTS="-j16 -l16"
EMERGE_DEFAULT_OPTS="--jobs 4 --load-average 16 --keep-going"

# --- USE ---
# 'lto' makes the packages that read the flag build with LTO.
# The stacked profile gives hardened, pic, xtpax and cet.
USE="lto"

# --- Licences ---
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"

# --- Branches ---
# The base system follows the stable branch. Packages that need the
# testing branch get an entry each in package.accept_keywords.
ACCEPT_KEYWORDS=""
```

`-O3` is a decision of this guide. Gentoo recommends `-O2` for general use. If
a package fails to build with `-O3`, lower the level for that package with
`/etc/portage/env`. Do not lower it for the full system.

`-ffast-math` is not in the global configuration, and it must stay out. It
relaxes guarantees of floating-point arithmetic that many packages depend on.
Apply it per package if you have a reason and you measured the result.

## Link-time optimisation

Gentoo supports LTO directly, thus the whole configuration lives in
`make.conf`. The `gentooLTO` overlay is in maintenance mode, and its own README
points at that same upstream support.

With Clang, the correct variant is ThinLTO:

```
-flto=thin
```

ThinLTO divides the work per translation unit and links in parallel. It thus
uses less time and less memory than monolithic LTO. In a large build, the
difference in memory decides if the machine completes the build or runs out of
RAM.

The warnings `-Werror=odr` and `-Werror=strict-aliasing` show problems that
appear only when you link with LTO. Note that Clang does not implement those
diagnostics completely. Their presence records the intention, and they will
protect you when the compiler emits them.

When a package fails to build with LTO, disable LTO for that package alone:

```bash
mkdir -p /etc/portage/env
cat > /etc/portage/env/no-lto <<'EOF'
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,--as-needed"
EOF

echo "category/package no-lto" >> /etc/portage/package.env/no-lto
```

## Rebuild the system

A change of profile and of flags needs a rebuild of each installed package.
The system is then consistent:

```bash
emerge --ask --verbose --update --deep --newuse @world
emerge --ask --emptytree @world
```

The second command builds each package again. This includes the packages that
keep their version. On a desktop machine it takes hours or days. The time
depends on the hardware and on the installed software. Start it when you can
leave the machine at work.

Then check that the hardening is in the binaries:

```bash
emerge app-misc/pax-utils
scanelf -e /usr/bin/emerge
checksec --file=/usr/bin/clang     # app-admin/checksec
```

## The kernel with Clang

The variables `LLVM=1` and `LLVM_IAS=1` build the kernel with LLVM. They select
Clang, LLD and the integrated assembler.

Attach that environment to the kernel sources:

```bash
mkdir -p /etc/portage/env /etc/portage/package.env
echo 'KBUILD_BUILD_ENV="LLVM=1 LLVM_IAS=1"' > /etc/portage/env/kernel-clang
echo "sys-kernel/* kernel-clang" >> /etc/portage/package.env/kernel
```

Then build:

```bash
cd /usr/src/linux
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

[installation.md](installation.md) gives the kernel configuration, the
initramfs and the bootloader.

---

> 🌐 **Language:** [Latina](../la/instrumenta.md) · [Español](../es/herramientas.md) · **English**
