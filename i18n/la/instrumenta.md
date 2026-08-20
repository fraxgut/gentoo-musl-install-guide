<!--
i18n/la/instrumenta.md
@guterion
CC-BY-SA-4.0
Toolchain: the LLVM/Clang profile, hardening and link-time optimisation
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/herramientas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/toolchain.md)**

# Instrumenta LLVM et LTO

</div>

---

Hoc documentum ostendit quomodo institutio musl, munitionem profili `hardened`
et instrumenta LLVM/Clang coniungat. Etiam ostendit quomodo optimationem in
tempore nectendi permittas.

## Index

1. [Problema profilorum](#problema-profilorum)
2. [Quid singula profila praebeant](#quid-singula-profila-praebeant)
3. [Profilum impositum](#profilum-impositum)
4. [Attritus notus](#attritus-notus)
5. [Compositio make.conf](#compositio-makeconf)
6. [Optimatio in tempore nectendi](#optimatio-in-tempore-nectendi)
7. [Systema reficere](#systema-reficere)
8. [Nucleus cum Clang](#nucleus-cum-clang)

## Problema profilorum

Gentoo profila separata pro musl cum munitione et pro musl cum LLVM praebet:

```
default/linux/amd64/23.0/musl/hardened   (exp)   →  GCC
default/linux/amd64/23.0/musl/llvm       (exp)   →  Clang, LLD, libc++
```

Nullum profilum officiale ea coniungit. Nullum stage3 `musl-llvm-hardened`
quoque est. Bibliotheca C et instrumenta compilandi figuntur cum stage3
expandis. Consilium ergo ante institutionem capis, neque id postea sine
refectione systematis mutare potes.

Hic libellus a stage3 `musl-llvm-openrc` incipit. Profilum `musl/llvm` eligit.
Deinde **proprietates profili `hardened` super id ponit** per profilum
proprium. Portage hanc compositionem permittit: profilum plures parentes
declarare potest, et ab omnibus hereditat.

Haec via exemplum habet. In foris Gentoo quidam ex tarball musl-clang instituit
et munitionem postea cum profilo proprio addidit. Wiki quoque eandem syntaxim
pro profilis LLVM scrinii aedificandis describit.

## Quid singula profila praebeant

Differentia minor est quam nomina suadent. Scire debes quid accipias et quid
maneat.

Profila 23.0 has proprietates praedefinite permittunt, **in omnibus casibus**:

- Programmata independentia a loco (PIE)
- Custodia acervi cum `-fstack-protector-strong`
- Custodia contra collisionem acervi
- RELRO et `BIND_NOW`

Profilum `hardened` haec addit:

| Res                                          | Fons                                      |
|----------------------------------------------|-------------------------------------------|
| `USE="hardened pic xtpax"`                    | `features/hardened/make.defaults`         |
| `USE="cet"` in amd64                          | `features/hardened/amd64/make.defaults`   |
| `PROFILE_IS_HARDENED=1`                       | `features/hardened/make.defaults`         |
| `pie` in `use.force`                          | `features/hardened/use.force`             |
| `sys-apps/elfix` in serie basali              | `features/hardened/packages`              |
| `xattr` coacta in tar, coreutils et portage   | `features/hardened/package.use.force`     |
| `-D_FORTIFY_SOURCE=3` pro 2                   | Instrumenta profili                       |
| Assertiones bibliothecae signatae             | Instrumenta profili                       |

Duas ultimas lineas impositio non praebet, quia ex compositione GCC veniunt et
non ex variabili profili. Utramque manu in `make.conf` ponis. Sectio infra
ostendit quomodo.

Una nota sincera de `xtpax`: vexillum programmata cum proprietatibus PaX
extensis notat, et idem profilum `PAX_MARKINGS="none"` ponit. Sine nucleo cum
PaX — et nullus nucleus publicus eum nunc habet — notae in tempore currendi
nihil mutant. Hic libellus vexillum servat quia ad seriem pertinet quae
profilum definit, et quia nihil constat. Defensio activa non est.

## Profilum impositum

Portage profilum in repositorio poscit. Locale fac:

```bash
emerge app-eselect/eselect-repository dev-vcs/git
eselect repository create local
```

Probā `metadata/layout.conf` formam profili declarare quae syntaxim
`repositorium:via` permittit. `eselect repository create` eam non semper
scribit:

```bash
cat /var/db/repos/local/metadata/layout.conf
```

```ini
masters = gentoo
profile-formats = portage-2
```

Profilum fac et parentes eius declara:

```bash
mkdir -p /var/db/repos/local/profiles/musl-llvm-hardened
echo 8 > /var/db/repos/local/profiles/musl-llvm-hardened/eapi

cat > /var/db/repos/local/profiles/musl-llvm-hardened/parent <<'EOF'
gentoo:default/linux/amd64/23.0/musl/llvm
gentoo:features/hardened
EOF
```

Ordo magni momenti est. Portage parentes ordine legit et variabiles
incrementales sicut `USE` accumulat. Ideo `features/hardened` vexilla sua
addit et vexilla profili LLVM servat. Variabiles quae incrementales non sunt
intactae manent. Eae sunt `CC`, `CXX`, `LD` et cetera instrumenta ex
`features/llvm`, quia profilum `hardened` ea non ponit.

Profilum registra, ut `eselect` id ostendat:

```bash
echo "$(portageq envvar ARCH) musl-llvm-hardened exp" \
    >> /var/db/repos/local/profiles/profiles.desc
```

Id elige:

```bash
eselect profile list
eselect profile set <numerus>
eselect profile show
```

Probā impositionem operari:

```bash
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

Primum mandatum quattuor vexilla ostendere debet. Secundum mandatum `clang`
imprimere debet.

## Attritus notus

`features/hardened/make.defaults` `USE="... -jit -orc"` ponit. In systemate cum
GCC illa vexilla fere nihil agunt. Hic compilatorem LLVM in tempore currendi et
stratum ORC eius claudunt, et quidam fasciculi ea poscunt. Mesa exemplum
usitatum est: pictor per programmatura et quaedam viae OpenCL JIT LLVM
adhibent.

Si fasciculus quem poscis illas facultates poscit, eas pro illo fasciculo
permitte. Ex profilo eas ne tollas:

```bash
mkdir -p /etc/portage/package.use
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

Ceteri fasciculi `-jit -orc` servant.

## Compositio make.conf

```ini
# /etc/portage/make.conf

# --- Instrumenta ---
# Profilum musl/llvm CC, CXX, LD et instrumenta LLVM ponit. Ea hic
# iterum declarare tantum repetit quod profilum praestat.

# --- Munitio ---
# Profilum 23.0 PIE, SSP, custodiam contra collisionem acervi, RELRO et
# BIND_NOW praebet. Hae duae macrones praebent quod profilum hardened ex
# compositione GCC accipit, et quod Clang non hereditat: gradum 3
# FORTIFY_SOURCE, et assertiones bibliothecae signatae C++.
HARDENING_FLAGS="-D_FORTIFY_SOURCE=3"
HARDENING_CXXFLAGS="-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST"

# --- Optimatio ---
# ThinLTO varietas LTO est quam Gentoo pro Clang commendat. In parallelo
# nectit, et multo minus memoriae adhibet quam LTO monolithicum.
# Monita quae errores fiunt discrepantias inter unitates translationis
# ostendunt. Illae difficultates tantum in tempore nectendi apparent.
WARNING_FLAGS="-Werror=odr -Werror=strict-aliasing"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${HARDENING_FLAGS} ${WARNING_FLAGS}"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS} ${HARDENING_CXXFLAGS}"
LDFLAGS="${COMMON_FLAGS} ${LDFLAGS}"

# --- Opera parallela ---
# Numerum cum exitu `nproc` supple. make.conf mandata non expandit.
MAKEOPTS="-j16 -l16"
EMERGE_DEFAULT_OPTS="--jobs 4 --load-average 16 --keep-going"

# --- USE ---
# 'lto' fasciculos qui vexillum legunt cum LTO construit.
# Profilum impositum hardened, pic, xtpax et cet praebet.
USE="lto"

# --- Licentiae ---
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"

# --- Rami ---
# Systema basale ramum stabilem sequitur. Fasciculi qui ramum probationis
# poscunt singulas voces in package.accept_keywords accipiunt.
ACCEPT_KEYWORDS=""
```

`-O3` consilium huius libelli est. Gentoo `-O2` pro usu communi commendat. Si
fasciculus cum `-O3` deficit, gradum pro illo fasciculo cum `/etc/portage/env`
demitte. Pro toto systemate eum ne demittas.

`-ffast-math` in compositione universali non est, et extra manere debet.
Cautiones arithmeticae fluitantis laxat quas multi fasciculi poscunt. Id per
fasciculos applica si causam habes et effectum mensus es.

## Optimatio in tempore nectendi

Gentoo LTO recta sustinet, ideo tota compositio in `make.conf` vivit. Overlay
`gentooLTO` in statu conservationis est, et ipsum README eius ad idem auxilium
Gentoo mittit.

Cum Clang varietas recta est ThinLTO:

```
-flto=thin
```

ThinLTO opus per unitatem translationis dividit et in parallelo nectit. Ideo
minus temporis et minus memoriae adhibet quam LTO monolithicum. In
constructione magna differentia memoriae decernit utrum machina opus perficiat
an memoriam consumat.

Monita `-Werror=odr` et `-Werror=strict-aliasing` difficultates ostendunt quae
tantum cum LTO nectis apparent. Nota Clang illa iudicia nondum plene facere.
Praesentia eorum consilium servat, et te custodient cum compilator ea emittet.

Cum fasciculus cum LTO deficit, LTO pro illo solo claude:

```bash
mkdir -p /etc/portage/env
cat > /etc/portage/env/no-lto <<'EOF'
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,--as-needed"
EOF

echo "categoria/fasciculus no-lto" >> /etc/portage/package.env/no-lto
```

## Systema reficere

Mutatio profili et vexillorum refectionem cuiusque fasciculi instituti poscit.
Systema tunc congruens est:

```bash
emerge --ask --verbose --update --deep --newuse @world
emerge --ask --emptytree @world
```

Secundum mandatum singulos fasciculos denuo construit. Hoc etiam fasciculos
comprehendit qui versionem suam servant. In machina scrinii horas vel dies
poscit. Tempus ex ferramentis et ex programmatura instituta pendet. Id incipe
cum machinam laborantem relinquere potes.

Deinde probā munitionem in programmatis esse:

```bash
emerge app-misc/pax-utils
scanelf -e /usr/bin/emerge
checksec --file=/usr/bin/clang     # app-admin/checksec
```

## Nucleus cum Clang

Variabiles `LLVM=1` et `LLVM_IAS=1` nucleum cum LLVM construunt. Clang, LLD et
assemblatorem inclusum eligunt.

Illum ambitum fontibus nuclei adiunge:

```bash
mkdir -p /etc/portage/env /etc/portage/package.env
echo 'KBUILD_BUILD_ENV="LLVM=1 LLVM_IAS=1"' > /etc/portage/env/kernel-clang
echo "sys-kernel/* kernel-clang" >> /etc/portage/package.env/kernel
```

Deinde construe:

```bash
cd /usr/src/linux
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

[institutio.md](institutio.md) compositionem nuclei, initramfs et oneratorem
initialem praebet.

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/herramientas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/toolchain.md)**

</div>
