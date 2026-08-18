<!--
i18n/la/optimatio.md
@fraxgut
CC-BY-SA-4.0
Optimisation tiers, hardware gates and the escape hatches that keep them safe
-->

# Optimatio

> 🌐 **Lingua:** **Latina** · [Español](../es/optimizacion.md) · [English](../en/optimisation.md)

Hic libellus fortiter optimat, et id intra ordinem fixum praelationum agit:

```
SECURITAS  >  MUNITIO  >  OPTIMATIO  >>>  CONSUETUDO
```

Tres prima inter se certant, et ordo decernit quod cedat. Quartum longe post
est. Gentoo `-O2` commendat, sed ea res `-O2` non imponit si melius quid
metimur. Vexillum insolitum non ideo falsum est quia insolitum est.

## Index

1. [Quid hic libellus servet](#quid-hic-libellus-servet)
2. [Gradus](#gradus)
3. [Gradus 0: securitas](#gradus-0-securitas)
4. [Gradus 1: fundamentum](#gradus-1-fundamentum)
5. [Gradus 2: acer](#gradus-2-acer)
6. [Gradus 3: per fasciculos](#gradus-3-per-fasciculos)
7. [Portae ferramentorum](#portae-ferramentorum)
8. [Distributor memoriae](#distributor-memoriae)
9. [Viae effugii](#viae-effugii)
10. [Nucleus conservativus](#nucleus-conservativus)
11. [Quid frangatur](#quid-frangatur)
12. [Quomodo metiaris](#quomodo-metiaris)

## Quid hic libellus servet

Compositiones acres quae in foris circumferuntur celeritatem plerumque emunt
cum defensiones claudunt. Exemplum notum `-hardened -ssp -seccomp -pie -pic` in
variabili USE ponit, et seriem vexillorum addit quae munitionem solvit:
`-U_FORTIFY_SOURCE`, `-fno-stack-protector`, `-fno-stack-clash-protection`,
`-fcf-protection=none`.

Hic libellus optimationes ex illis compositionibus sumit, et defensiones
servat:

| Defensio                       | Status hic | Quid constet servare                       |
|--------------------------------|------------|--------------------------------------------|
| `USE="hardened pic"`           | Aperta     | Fere nihil in usu.                         |
| PIE et `-fPIC`                 | Apertae    | Unum receptaculum in x86-64. Vix mensurabile. |
| `-fstack-protector-strong`     | Aperta     | Unum prooemium in quaque functione cum seriebus. |
| `-fstack-clash-protection`     | Aperta     | Tentamina acervi in functionibus magnis.   |
| `-D_FORTIFY_SOURCE=3`          | Aperta     | Probationes magnitudinis quas compilator in tempore construendi solvit cum potest. |
| `-fcf-protection=full`, `USE="cet"` | Apertae | Mandata ENDBR. Strepitus in magnitudine.  |
| `USE="seccomp"`                | Aperta     | Fere nihil.                                |
| Assertiones libc++             | Apertae    | Probationes condicionum in receptaculis.   |

Mensura hanc electionem sustinet, et ex eadem disputatione venit quae
solutionem munitionis proponit. Unus particeps vexilla singillatim cum
Phoronix Test Suite mensus est. Magna pars effectuum intra marginem erroris
mansit. De `_FORTIFY_SOURCE` scripsit se id denuo aperire cum differentiam non
invenit, et ita accidisse. Exemplum quoque dedit programmatis in quo
`_FORTIFY_SOURCE` amissiones memoriae veras et exundationes acervi prohibuit.

Alius particeps rem incommodiorem de solutione munitionis notavit. Vexilla USE
`-pie -pic` tantum fasciculos mutant qui ea declarant, ideo ceteri fasciculi
cum `-fPIC` nihilominus construuntur. Defensio ex parte perit, et celeritas non
advenit.

Una correctio nominum, si ex illis disputationibus venis: **`extra-hardened`
vexillum USE universale non est.** In arbore Gentoo tantum in
`app-admin/clsync` exsistit, et ibi contrarium significat eius quod nomen in
illo contextu suadet: plures probationes securitatis aperit et efficientiam
constat.

## Gradus

Compositio quattuor gradus habet. Cum quid deficit, gradum altissimum primum
laxa et descende. Numquam contrarium.

| Gradus | Contentum                                      | Quando laxes |
|--------|------------------------------------------------|--------------|
| **0**  | Securitas: subscriptiones, seccomp, facultates | Numquam      |
| **1**  | Fundamentum: `-O3`, `-march=native`, ThinLTO   | Ultimum      |
| **2**  | Acer: arithmetica celeris, evolutio, Polly     | Primum       |
| **3**  | Per fasciculos: PGO, distributores, jumbo-build | Singillatim |

## Gradus 0: securitas

Hic gradus cum efficientia non paciscitur, quia cum ea non certat.

```ini
USE="verify-sig seccomp caps filecaps"
```

`verify-sig` subscriptiones plicarum quas Portage deprömit probat. Defensio
vilissima systematis est. Fiduciam ab «speculum mihi quid dedit» ad
«evolutor hoc subscripsit» movet.

`seccomp` vocationes systematis in tempore currendi cribrat. Pretium eius fere
nihil est in plerisque casibus, et id claudere effectus graves habet: in
receptaculis maiorem partem seclusionis tollit.

`caps` et `filecaps` programmata cum bit setuid facultatibus certis supplent.
Programma quod portum humilem aperire debet illam facultatem accipit, neque
radix plena fit.

Probationem subscriptionum pro ipsa arbore Portage quoque adde:

```ini
FEATURES="${FEATURES} webrsync-gpg"
```

## Gradus 1: fundamentum

Hic gradus toti systemati applicatur.

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
```

`-O3` contra `-O2` consilium deliberatum est. Distantia inter ea minor facta
est cum `-ftree-vectorize` ex `-O3` ad `-O2` migravit, et in multis
programmatis differentia intra strepitum cadit. `-O3` servamus quia pretium
tempus construendi est, et tempus construendi moneta est quam haec institutio
iam impendere consensit.

`-march=native` processorem accuratius describit quam ullum nomen familiae.

ThinLTO opus per unitatem translationis dividit et in parallelo nectit. Una
nota sincera: LTO monolithicum codicem aliquanto meliorem facit, et ThinLTO
parallelismum et memoriam illa differentia emit. In machina quae LibreOffice
construit, memoria decernit utrum nexus perficiatur.

## Gradus 2: acer

Hic est gradus quem primum laxas.

```ini
# Arithmetica celeris quae NaN et infinitum servat
FASTMATH_FLAGS="-ffast-math -fno-finite-math-only"

# Dispositio codicis
FUN_FLAGS="-funroll-loops -fno-plt -fno-semantic-interposition \
           -fdata-sections -ffunction-sections \
           -falign-functions=${CACHELINE}"

# Polly, aequivalens Graphite apud LLVM
POLLY_FLAGS="-mllvm=-polly -mllvm=-polly-vectorizer=stripmine \
             -mllvm=-polly-invariant-load-hoisting"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${FASTMATH_FLAGS} ${FUN_FLAGS}"
LDFLAGS="${COMMON_FLAGS} -Wl,-O2 -Wl,--as-needed -Wl,--gc-sections \
         -Wl,-z,relro -Wl,-z,now -Wl,-z,pack-relative-relocs"
```

Quattuor notae de his vexillis.

**`-ffast-math` pro `-Ofast` adhibe.** Clang `-Ofast` obsoletum fecit et id hac
ipsa expansione supplet. Systema huius libelli cum Clang construitur, ideo eam
formam scribimus quam Clang servat.

**`-fno-finite-math-only` est quod `-ffast-math` tolerabile facit.**
Tractationem NaN et infiniti restituit, quae fons usitatus fractionum est.
Nihilominus `-ffast-math` reassociationem permittit. Si programma tuum ab
effectu exacto arithmeticae fluitantis pendet, id metire antequam ei confidis.

**`-funroll-loops` a codice pendet.** A geometria memoriae celeris, a
mitigationibus nuclei activis et ab ipso programmate pendet. Quosdam casus
meliores facit et alios peiores, ideo ad gradum pertinet quem metiris, non ad
gradum quem praesumis.

**Polly syntaxi `-mllvm=` cum signo aequalitatis utitur.** Forma separata
`-mllvm -polly` constructionem quorundam fasciculorum magnorum frangit. Polly
per `llvm-runtimes/clang-runtime` cum `USE="polly"` advenit, et descriptio
vexilli dicit te deinde `-mllvm -polly` tradere debere ut eo utaris. Id pro
omnibus fasciculis ne permittas, quia plures deficiunt.

`-fno-semantic-interposition` notam poscit. Codicem PIE optimat, ideo hic
prodest quia hoc systema programmata independentia a loco construit. In
systemate sine munitione et sine PIE nihil ageret.

Graphite extra manet. Ad GCC pertinet et Clang id non habet. Tantum
fasciculis applicaretur qui ad viam effugii cum GCC cadunt.

## Gradus 3: per fasciculos

Haec vexilla universalia non sunt, et arbor loca eorum artat.

| Vexillum USE        | Fasciculi qui id praebent | Quid agat                                 |
|---------------------|---------------------------|-------------------------------------------|
| `pgo`               | 11                        | Construit, profilum currit, iterum construit. |
| `jumbo-build`       | 5                         | **Constructionem** accelerat, non programma. |
| `native-extensions` | 14                        | Extensiones nativas pro codice puro construit. |
| `custom-cflags`     | 1                         | Prohibet fasciculum tua CFLAGS abicere.   |

`pgo` in `bash`, `python`, `xz-utils`, `binutils`, `gcc`, `firefox`,
`thunderbird`, `chromium`, `svt-av1` et duobus aliis est. Optimam rationem
inter lucrum et periculum in hac pagina habet, quia programma verum metitur
loco coniecturae. Duplex tempus construendi constat.

```bash
echo "dev-lang/python pgo" >> /etc/portage/package.use/optimatio
echo "app-shells/bash pgo"  >> /etc/portage/package.use/optimatio
echo "app-arch/xz-utils pgo" >> /etc/portage/package.use/optimatio
```

`jumbo-build` plicas fontium coniungit ut celerius construat, et plus memoriae
adhibet. Programma non mutat, ideo ad seriem commoditatum pertinet et non ad
seriem efficientiae.

**BOLT vexillum USE in arbore non habet.** Si id vis, opus manuale in
programmate iam nexo est.

## Portae ferramentorum

Quaedam vexilla a processore pendent. Machinam de responso roga, ne id ex foro
transcribas.

### Adaequatio functionum

Clear Linux `-falign-functions=32` popularem fecit, et ille numerus tantum in
processoribus Intel a Sandy Bridge prodest. Regula recta functiones ad
magnitudinem lineae memoriae celeris mandatorum adaequat, quam systema nuntiat:

```bash
getconf -a | grep LEVEL1_ICACHE_LINESIZE
```

```
LEVEL1_ICACHE_LINESIZE              64
```

In illa machina responsum est `-falign-functions=64`. Valorem lege antequam
eligis:

```bash
export CACHELINE=$(getconf LEVEL1_ICACHE_LINESIZE)
echo "-falign-functions=${CACHELINE}"
```

### Quid `-march=native` in machina tua significet

Inveni quid revera aperiat, praesertim si pro alia machina construis:

```bash
gcc -march=native -E -v - </dev/null 2>&1 | grep cc1
clang -march=native -E -v - </dev/null 2>&1 | grep cc1
```

`app-misc/resolve-march-native` idem legibiliter ostendit. Exitus magnitudines
memoriae celeris continet quas compilator ad consilia sua adhibet, et
compositionem in alia machina reddere permittit.

### Vexilla processoris pro Portage

```bash
emerge app-portage/cpuid2cpuflags
echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
```

## Distributor memoriae

Hoc plus hic valet quam in institutione cum glibc, et attentionem accipit
praecise quia systema musl adhibet.

Distributor musl magnitudini codicis et resistentiae contra fragmentationem
praelationem dat, supra celeritatem nudam. In opere quod multum memoriae
distribuit et liberat, distributor alius melius agit. Qui distributores
comparant nuntiant jemalloc plerumque eum glibc superare, et distantiam ab eo
musl maiorem adhuc esse.

Gentoo electionem per fasciculos praebet:

```ini
jemalloc  - Use dev-libs/jemalloc for memory management
tcmalloc  - Use the dev-util/google-perftools libraries to replace malloc()
```

Fere nullus fasciculus utrumque accipit, ideo unum pro quoque fasciculo eligis:

```bash
echo "dev-db/redis jemalloc" >> /etc/portage/package.use/allocator
```

mimalloc quoque in arbore pro quibusdam fasciculis est. Unum monitum certum ab
eo qui contra id nectit: `bash` frangit. Si id temptas, per fasciculos age et
extra `LDFLAGS` universalia serva.

## Viae effugii

Systema optimatum viam ordinatam poscit ut in casibus certis cedat.
`/etc/portage/env` profila tenet et `package.env` ea assignat.

Haec quattuor fere omnia comprehendunt:

```bash
mkdir -p /etc/portage/env
```

`/etc/portage/env/safest.conf` — deditio plena, pro eo quod aliter non
construitur:

```ini
CC="gcc"
CXX="g++"
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-fuse-ld=bfd -Wl,-O1 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-lto.conf` — optimationes servat et LTO dimittit:

```ini
COMMON_FLAGS="-O3 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,-O2 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-fastmath.conf` — pro fasciculis qui arithmeticam celerem
recusant:

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
```

`/etc/portage/env/heavy.conf` — pro nexibus qui omnem memoriam adhibent:

```ini
MAKEOPTS="-j1 -l2"
NINJAOPTS="-j1 -l2"
```

Ea sic assigna:

```bash
mkdir -p /etc/portage/package.env
echo "dev-lang/python no-fastmath.conf" >> /etc/portage/package.env/10-toolchain
echo "dev-qt/qtwebengine heavy.conf"    >> /etc/portage/package.env/20-desktop
```

Unus error frequens in his plicis: si `COMMON_FLAGS` ponis et deinde `CFLAGS`
ex alia variabili assignas, fasciculus sine `-march` et sine `-O` construitur.
Probā quid accipiat:

```bash
emerge --info categoria/fasciculus | grep -E '^(CFLAGS|CXXFLAGS|LDFLAGS)'
```

## Nucleus conservativus

Qui talia systemata curant consentiunt quae strata primum frangantur:
instrumenta compilandi, bibliotheca C, bibliothecae graphicae infimae et
bibliothecae cryptographicae.

Hoc initium rationabile est. Systema incipiens servat dum cetera vexilla acria
accipiunt:

```
# Instrumenta compilandi et bibliotheca C
sys-devel/binutils      safest.conf
sys-devel/gcc           safest.conf
sys-libs/musl           safest.conf
sys-kernel/linux-headers safest.conf
llvm-core/clang         safest.conf heavy.conf
llvm-core/llvm          safest.conf heavy.conf

# Cryptographia et rete
dev-libs/openssl        safest.conf
net-misc/curl           safest.conf
sys-libs/zlib           safest.conf

# Graphica
media-libs/mesa         safest.conf
x11-libs/libX11         safest.conf
x11-libs/libxcb         safest.conf
media-libs/freetype     safest.conf
media-libs/harfbuzz     safest.conf
```

Ratio periculum est, non efficientia. Vitium subtile in `openssl` vel in `musl`
non ut error constructionis apparet. Ut data corrupta apparet, vel ut probatio
cryptographica quae transit cum deficere debet. Hae partes ex optimatione acri
parum lucrantur, et fere omne damnum possibile tenent.

## Quid frangatur

Haec series certa est, ab eis qui iam temptaverunt.

**Haec cum arithmetica celeri non construuntur:**

```
dev-lang/python
net-libs/nodejs
```

**Haec `-ffinite-math-only` recusant:**

```
sys-apps/systemd-utils
sys-auth/polkit
media-libs/opus
dev-lang/duktape
sys-auth/elogind
```

Hi fasciculi difficultatem inter constructionem nuntiant, quod modus benignus
deficiendi est. Casus malus contrarius est: fasciculus vexilla accipit et
programma facit quod postea cum vitio segmentationis deficit. Series
probationum quosdam eorum prius invenit:

```ini
FEATURES="${FEATURES} test"
```

Non omnes invenit. Id scias antequam decernis quousque gradum 2 feras.

## Quomodo metiaris

Compositio acris sine mensura opinio est. Ratio parva sed severa est.

```bash
emerge app-benchmarks/hyperfine
```

Onus verum compara, non circulum fictum:

```bash
hyperfine --warmup 3 --runs 9 \
  'xz -9 -T1 -c /usr/portage/distfiles/exemplum.tar > /dev/null'
```

Tres regulae quae conclusiones falsas prohibent:

**Novem cursus ad minimum.** Comparatio trium cursuum emendationem decem
centesimarum fabricat quam plura exempla dissolvunt.

**Scias quid onus arceat antequam lucrum promittis.** Analysis textus,
persecutio indicum et comparatio catenarum ex vectoribus latioribus nihil
lucrantur. Arithmetica fluitans densa lucratur.

**Effectum sincerum nuntia,** et responsum «haec vexilla hic nihil emerunt»
include. Illud responsum frequentissimum est, et refectiones parcit.

Alteram partem librae quoque metire. Tempus construendi pretium est quod haec
vexilla exigunt, et `genlop` id ex actis Portage legit.

```bash
emerge app-portage/genlop
genlop -t categoria/fasciculus
```

---

> 🌐 **Lingua:** **Latina** · [Español](../es/optimizacion.md) · [English](../en/optimisation.md)
