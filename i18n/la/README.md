<!--
i18n/la/README.md
@guterion
CC-BY-SA-4.0
Latin landing page and index for the installation guide
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/README.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/README.md)**

<a href="https://www.gentoo.org/"><img src="../../assets/gentoo-logo.png" alt="Gentoo Linux logo" width="72" height="72"></a>

# Index institutionis Gentoo Linux provectae

**Institutio systematis Gentoo Linux in architectura AMD64 cum musl, OpenRC,
LUKS2, Btrfs, LLVM/Clang, ThinLTO et nucleo Zen**

</div>

---

## De hoc libello

Hic libellus unam certam compositionem describit, non institutionem Gentoo
communem. Singulae partes ex consilio explicito oriuntur, et textus causam
simul cum ratione agendi praebet.

Ad eos scribitur qui iam Gentoo noverunt. Praesumit te `emerge`, vexilla USE,
profila et nuclei compositionem manualem tenere.

## Compositio proposita

| Pars                    | Electio                                       |
|-------------------------|-----------------------------------------------|
| Architectura            | AMD64                                         |
| Bibliotheca C           | musl                                          |
| Systema initiale        | OpenRC                                        |
| Encryptio               | LUKS2 cum Argon2id                            |
| Systema plicarum        | Btrfs cum subvolumnibus                       |
| Instrumenta compilandi  | Clang, LLD et libc++                          |
| Optimatio               | ThinLTO                                       |
| Nucleus                 | Zen                                           |
| Munitio                 | Profilum `hardened` super `musl/llvm` impositum |
| Volumina logica         | LVM ut varietas optionalis                    |

## Documenta

1. [Institutio](institutio.md) — a medio institutionis usque ad primum initium.
2. [Receptaculum](receptaculum.md) — partitiones, LUKS2, Btrfs, compressio, TRIM et permutatio.
3. [Instrumenta](instrumenta.md) — profilum impositum, munitio et optimatio in tempore nectendi.
4. [Optimatio](optimatio.md) — gradus, portae ferramentorum et viae effugii.
5. [Remedia](remedia.md) — restitutio secundum signum.

## Monita

Arbor Gentoo profila `musl/llvm` et `musl/hardened` experimentalia notat.
Programmatura quae cum GCC et glibc compilatur non semper cum Clang et musl
compilatur.

Bibliotheca C, instrumenta compilandi et systema initiale figuntur cum stage3
expandis. Ut ea postea mutes, totum systema reficere debes. Consilium ergo ante
institutionem capis.

Rationes partitionum data delent. Exemplaria subsidiaria fac, et nomen
apparatus bis inspice.

## Iter propositum

- [x] Btrfs super LUKS2 cum subvolumnibus
- [x] musl cum OpenRC
- [x] Clang, LLD et libc++ ex stage3 officiali
- [x] Munitio super profilum LLVM imposita
- [x] ThinLTO cum auxilio Gentoo
- [x] Nucleus Zen cum LLVM constructus
- [x] Permutatio nativa Btrfs cum hibernatione
- [ ] Secure Boot, ad catenam initialem extra LUKS authenticandam
- [ ] SELinux
- [ ] Exemplum compositionis nuclei
- [ ] Imagines momentaneae automaticae et initium ex imagine

## Licentia

Nisi aliter notatur, haec documenta sub licentia Creative Commons
Attributio-CondicionibusIisdem 4.0 Internationali (CC BY-SA 4.0) divulgantur.
Vide [LICENCE.md](../../LICENCE.md).

Nomen et insigne Gentoo ad Gentoo Foundation pertinent. Haec licentia ea non
comprehendit. Usus eorum [Gentoo Name and Logo Usage
Guidelines](https://www.gentoo.org/inside-gentoo/foundation/name-logo-guidelines.html)
sequitur.

Hic libellus opus privatum est. Pro auctore solo loquitur, neque Gentoo
Foundation in eo partem habet.

## De conferendo

Collationes gratae sunt. [CONTRIBUTING.md](../../CONTRIBUTING.md) consuetudines
commissionum et regulas quibus translationes inter se congruunt praebet.

## Ad quem scribas

Franco Gutiérrez — [@guterion](https://github.com/guterion) — franco.gutierrez.1@ug.uchile.cl

## Fontes

- [Manuale Gentoo pro AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Inceptum musl apud Gentoo](https://wiki.gentoo.org/wiki/Project:Musl)
- [Profila propria in Portage](https://wiki.gentoo.org/wiki/Portage/Profiles/Custom_profiles)
- [LLVM apud Gentoo](https://wiki.gentoo.org/wiki/LLVM)
- [LTO apud Gentoo](https://wiki.gentoo.org/wiki/LTO)
- [Documenta Btrfs](https://btrfs.readthedocs.io/)
- [Encryptio totius disci cum dm-crypt](https://wiki.gentoo.org/wiki/Dm-crypt_full_disk_encryption)
- [Dracut apud Gentoo](https://wiki.gentoo.org/wiki/Dracut)

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/README.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/README.md)**

</div>
