<!--
i18n/en/optimisation.md
@guterion
CC-BY-SA-4.0
Optimisation tiers, hardware gates and the escape hatches that keep them safe
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/optimatio.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/optimizacion.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **English**

# Optimisation

</div>

---

This guide optimises hard, and it does that inside a fixed order of priorities:

```
SAFETY  >  HARDENING  >  OPTIMISATION  >>>  CONVENTIONALITY
```

The first three terms compete with each other, and the order decides which one
gives way. The fourth is far behind. Gentoo recommends `-O2`, but that is no
reason to use `-O2` if we measure something better. An unusual flag is not
wrong because it is unusual.

## Contents

1. [What this guide keeps on](#what-this-guide-keeps-on)
2. [The tiers](#the-tiers)
3. [Tier 0: safety](#tier-0-safety)
4. [Tier 1: the base](#tier-1-the-base)
5. [Tier 2: aggressive](#tier-2-aggressive)
6. [Tier 3: per package](#tier-3-per-package)
7. [Hardware gates](#hardware-gates)
8. [The memory allocator](#the-memory-allocator)
9. [The escape hatches](#the-escape-hatches)
10. [The conservative core](#the-conservative-core)
11. [What breaks](#what-breaks)
12. [How to measure](#how-to-measure)

## What this guide keeps on

The aggressive configurations that circulate in forums usually buy speed when
they turn defences off. One well known example puts `-hardened -ssp -seccomp
-pie -pic` in the USE variable, and it adds a block of flags that undoes the
hardening: `-U_FORTIFY_SOURCE`, `-fno-stack-protector`,
`-fno-stack-clash-protection`, `-fcf-protection=none`.

This guide takes the optimisations from those configurations, and it keeps the
defences:

| Defence                        | State here | What it costs to keep                     |
|--------------------------------|------------|-------------------------------------------|
| `USE="hardened pic"`           | On         | Almost nothing in practice.               |
| PIE and `-fPIC`                | On         | One register on x86-64. Hard to measure.  |
| `-fstack-protector-strong`     | On         | One prologue in each function with arrays. |
| `-fstack-clash-protection`     | On         | Stack probes in large functions.          |
| `-D_FORTIFY_SOURCE=3`          | On         | Size checks that the compiler solves at build time when it can. |
| `-fcf-protection=full`, `USE="cet"` | On    | ENDBR instructions. Noise in the size.    |
| `USE="seccomp"`                | On         | Almost zero.                              |
| libc++ assertions              | On         | Precondition checks in the containers.    |

There is a measurement that supports this choice, and it comes from the same
discussion that proposes the dehardening. One participant measured the flags
one by one with the Phoronix Test Suite. Much of the result stayed inside the
margin of error. About `_FORTIFY_SOURCE` he wrote that he turned it back on
when he found no difference, and that this was the case. He also gave one
example of a program where `_FORTIFY_SOURCE` stopped real memory leaks and
heap overflows.

Another participant made a more uncomfortable point about the dehardening. The
USE flags `-pie -pic` change only the packages that declare them, thus the
other packages still build with `-fPIC`. The defence goes away in part, and the
speed does not arrive.

One correction of names, if you come from those discussions:
**`extra-hardened` is not a global USE flag.** In the Gentoo tree it exists
only in `app-admin/clsync`, and there it means the opposite of what the name
suggests in that context: it turns on more security checks and costs
performance.

## The tiers

The configuration has four tiers. When something fails, relax the highest
number first and go down. Never go the other way.

| Tier   | Content                                        | Relax it   |
|--------|------------------------------------------------|------------|
| **0**  | Safety: signatures, seccomp, capabilities      | Never      |
| **1**  | Base: `-O3`, `-march=native`, ThinLTO          | Last       |
| **2**  | Aggressive: fast math, unrolling, Polly        | First      |
| **3**  | Per package: PGO, allocators, jumbo-build      | One by one |

## Tier 0: safety

This tier does not negotiate with performance, because it does not compete
with it.

```ini
USE="verify-sig seccomp caps filecaps"
```

`verify-sig` checks the signatures of the files that Portage downloads. It is
the cheapest defence in the system. It moves the trust from "the mirror gave me
something" to "the developer signed this".

`seccomp` filters system calls at run time. Its cost is almost zero in most
cases, and to turn it off has serious results: in containers it removes most of
the isolation.

`caps` and `filecaps` replace setuid binaries with specific capabilities. A
program that must open a low port gets that capability, and it does not become
full root.

Add the signature check for the Portage tree as well:

```ini
FEATURES="${FEATURES} webrsync-gpg"
```

## Tier 1: the base

This tier applies to the whole system.

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
```

`-O3` against `-O2` is a deliberate choice. The distance between them became
smaller when `-ftree-vectorize` moved from `-O3` to `-O2`, and in many programs
the difference falls inside the noise. We keep `-O3` because the cost is build
time, and build time is the currency that this installation already agreed to
spend.

`-march=native` describes the processor with more detail than any family name.

ThinLTO divides the work per translation unit and links in parallel. One honest
note: monolithic LTO produces somewhat better code, and ThinLTO buys
parallelism and memory with that difference. On a machine that builds
LibreOffice, the memory decides if the link completes.

## Tier 2: aggressive

This is the tier that you relax first.

```ini
# Fast math that keeps NaN and infinity
FASTMATH_FLAGS="-ffast-math -fno-finite-math-only"

# Code layout
FUN_FLAGS="-funroll-loops -fno-plt -fno-semantic-interposition \
           -fdata-sections -ffunction-sections \
           -falign-functions=${CACHELINE}"

# Polly, the LLVM equivalent of Graphite
POLLY_FLAGS="-mllvm=-polly -mllvm=-polly-vectorizer=stripmine \
             -mllvm=-polly-invariant-load-hoisting"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${FASTMATH_FLAGS} ${FUN_FLAGS}"
LDFLAGS="${COMMON_FLAGS} -Wl,-O2 -Wl,--as-needed -Wl,--gc-sections \
         -Wl,-z,relro -Wl,-z,now -Wl,-z,pack-relative-relocs"
```

Four notes on these flags.

**Use `-ffast-math` in place of `-Ofast`.** Clang made `-Ofast` obsolete and
replaces it with exactly this expansion. The system of this guide builds with
Clang, thus we write the form that Clang keeps.

**`-fno-finite-math-only` is what makes `-ffast-math` tolerable.** It restores
the handling of NaN and infinity, which is the usual source of breakage. Even
so, `-ffast-math` permits reassociation. If a program of yours depends on the
exact floating-point result, measure it before you trust it.

**`-funroll-loops` is sensitive to the code.** It depends on the cache
geometry, on the active kernel mitigations and on the program. It improves some
cases and it makes others worse, thus it belongs in the tier that you measure,
not in the tier that you assume.

**Polly uses the `-mllvm=` syntax, with the equals sign.** The separate form
`-mllvm -polly` breaks the build of some large packages. Polly arrives through
`llvm-runtimes/clang-runtime` with `USE="polly"`, and the description of the
flag says that you must then pass `-mllvm -polly` to use it. Do not enable it
for all packages, because several fail.

`-fno-semantic-interposition` needs a note. It optimises PIE code, thus it
helps here precisely because this system builds position independent
executables. On a dehardened system without PIE it does nothing.

Graphite stays out. It belongs to GCC and Clang does not have it. It would
apply only to the packages that fall through to the GCC escape hatch.

## Tier 3: per package

These flags are not global, and the tree limits where they exist.

| USE flag            | Packages that offer it | What it does                              |
|---------------------|------------------------|-------------------------------------------|
| `pgo`               | 11                     | Builds, runs a profile, and builds again. |
| `jumbo-build`       | 5                      | Speeds up the **build**, not the binary.  |
| `native-extensions` | 14                     | Builds native extensions in place of pure code. |
| `custom-cflags`     | 1                      | Stops the package from discarding your CFLAGS. |

`pgo` exists in `bash`, `python`, `xz-utils`, `binutils`, `gcc`, `firefox`,
`thunderbird`, `chromium`, `svt-av1` and two more. It has the best ratio of
gain to risk on this page, because it measures the real program instead of a
guess. It costs twice the build time.

```bash
echo "dev-lang/python pgo" >> /etc/portage/package.use/optimisation
echo "app-shells/bash pgo"  >> /etc/portage/package.use/optimisation
echo "app-arch/xz-utils pgo" >> /etc/portage/package.use/optimisation
```

`jumbo-build` combines source files to build faster, and it uses more memory.
It does not change the binary, thus it belongs with the conveniences and not
with the performance flags.

**BOLT has no USE flag in the tree.** If you want it, it is manual work on the
linked binary.

## Hardware gates

Some flags depend on the processor. Ask the machine for the answer instead of a
copy from a forum.

### Function alignment

Clear Linux made `-falign-functions=32` popular, and that number helps only on
Intel processors from Sandy Bridge. The correct rule aligns functions to the
size of the instruction cache line, which the system reports:

```bash
getconf -a | grep LEVEL1_ICACHE_LINESIZE
```

```
LEVEL1_ICACHE_LINESIZE              64
```

On that machine the answer is `-falign-functions=64`. Read the value before you
choose:

```bash
export CACHELINE=$(getconf LEVEL1_ICACHE_LINESIZE)
echo "-falign-functions=${CACHELINE}"
```

### What `-march=native` means on your machine

Find what it turns on, above all if you build for a different machine:

```bash
gcc -march=native -E -v - </dev/null 2>&1 | grep cc1
clang -march=native -E -v - </dev/null 2>&1 | grep cc1
```

`app-misc/resolve-march-native` shows the same result in a readable form. The
output holds the cache sizes that the compiler uses for its decisions, and it
lets you reproduce the configuration on a different machine.

### CPU flags for Portage

```bash
emerge app-portage/cpuid2cpuflags
echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
```

## The memory allocator

This point counts for more here than on a glibc installation, and it gets
attention precisely because the system uses musl.

The musl allocator gives priority to code size and to resistance against
fragmentation, above raw speed. In work that allocates and frees much memory,
a different allocator performs better. People who compare allocators report
that jemalloc usually beats the glibc one, and that the distance from the musl
one is larger still.

Gentoo offers the choice per package:

```ini
jemalloc  - Use dev-libs/jemalloc for memory management
tcmalloc  - Use the dev-util/google-perftools libraries to replace malloc()
```

Almost no package accepts both, thus you select one for each package:

```bash
echo "dev-db/redis jemalloc" >> /etc/portage/package.use/allocator
```

mimalloc is also in the tree for some packages. One concrete warning from a
person who links against it: it breaks `bash`. If you try it, do that per
package and keep it out of the global `LDFLAGS`.

## The escape hatches

An optimised system needs an orderly way to give up in specific cases.
`/etc/portage/env` holds the profiles and `package.env` assigns them.

These four cover almost everything:

```bash
mkdir -p /etc/portage/env
```

`/etc/portage/env/safest.conf` — the full surrender, for what builds no other
way:

```ini
CC="gcc"
CXX="g++"
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-fuse-ld=bfd -Wl,-O1 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-lto.conf` — keeps the optimisations and releases LTO:

```ini
COMMON_FLAGS="-O3 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,-O2 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-fastmath.conf` — for the packages that refuse fast math:

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
```

`/etc/portage/env/heavy.conf` — for the links that use all the memory:

```ini
MAKEOPTS="-j1 -l2"
NINJAOPTS="-j1 -l2"
```

Assign them like this:

```bash
mkdir -p /etc/portage/package.env
echo "dev-lang/python no-fastmath.conf" >> /etc/portage/package.env/10-toolchain
echo "dev-qt/qtwebengine heavy.conf"    >> /etc/portage/package.env/20-desktop
```

One frequent error in these files: if you set `COMMON_FLAGS` and then assign
`CFLAGS` from a different variable, the package builds without `-march` and
without `-O`. Check what it receives:

```bash
emerge --info category/package | grep -E '^(CFLAGS|CXXFLAGS|LDFLAGS)'
```

## The conservative core

The people who keep such systems agree on which layers break first: the
toolchain, the C library, the low level graphics libraries and the
cryptography libraries.

This is a reasonable start. It keeps the system bootable while the rest gets
the aggressive flags:

```
# Toolchain and C library
sys-devel/binutils      safest.conf
sys-devel/gcc           safest.conf
sys-libs/musl           safest.conf
sys-kernel/linux-headers safest.conf
llvm-core/clang         safest.conf heavy.conf
llvm-core/llvm          safest.conf heavy.conf

# Cryptography and network
dev-libs/openssl        safest.conf
net-misc/curl           safest.conf
sys-libs/zlib           safest.conf

# Graphics
media-libs/mesa         safest.conf
x11-libs/libX11         safest.conf
x11-libs/libxcb         safest.conf
media-libs/freetype     safest.conf
media-libs/harfbuzz     safest.conf
```

The logic is risk, not performance. A subtle fault in `openssl` or in `musl`
does not show as a build error. It shows as corrupt data, or as a cryptographic
check that passes when it must fail. These parts get little benefit from
aggressive optimisation, and they hold almost all the possible damage.

## What breaks

This is a specific list from people who tried it.

**These do not build with fast math:**

```
dev-lang/python
net-libs/nodejs
```

**These refuse `-ffinite-math-only`:**

```
sys-apps/systemd-utils
sys-auth/polkit
media-libs/opus
dev-lang/duktape
sys-auth/elogind
```

These packages report the problem during the build, which is the kind way to
fail. The bad case is the opposite one: a package accepts the flags and makes a
binary that fails later with a segmentation fault. The test suite finds some of
them first:

```ini
FEATURES="${FEATURES} test"
```

It does not find them all. Know that before you decide how far to take tier 2.

## How to measure

An aggressive configuration without measurement is a belief. The protocol is
small but strict.

```bash
emerge app-benchmarks/hyperfine
```

Compare a real workload, not a synthetic loop:

```bash
hyperfine --warmup 3 --runs 9 \
  'xz -9 -T1 -c /usr/portage/distfiles/sample.tar > /dev/null'
```

Three rules that prevent false conclusions:

**Nine runs as a minimum.** A comparison of three runs manufactures a ten per
cent improvement that more samples dissolve.

**Know what limits the workload before you promise a gain.** Text parsing,
pointer chasing and string comparison gain nothing from wider vectors. Dense
floating-point arithmetic does.

**Report the honest result,** and include the answer "these flags bought
nothing here". That is the most frequent answer, and it saves rebuilds.

Measure the other side of the balance as well. Build time is the price that
these flags collect, and `genlop` reads it from the Portage log.

```bash
emerge app-portage/genlop
genlop -t category/package
```

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/optimatio.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/optimizacion.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **English**

</div>
