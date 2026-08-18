<!--
README.md
@fraxgut
CC-BY-SA-4.0
Project landing page and language selector
-->

<div align="center">

<a href="https://www.gentoo.org/"><img src="assets/gentoo-logo.png" alt="Gentoo Linux logo" width="80" height="80"></a>

# Gentoo musl Installation Guide

**Advanced Gentoo Linux installation for AMD64 musl systems**

OpenRC · LUKS2 · Btrfs · LLVM/Clang · ThinLTO · Zen kernel

<a href="LICENCE.md"><img src="https://img.shields.io/badge/licence-CC%20BY--SA%204.0-%2362A98B?style=for-the-badge" alt="Licence: CC BY-SA 4.0"/></a>
<img src="https://img.shields.io/badge/architecture-AMD64-informational?style=for-the-badge" alt="Architecture: AMD64"/>
<img src="https://img.shields.io/badge/libc-musl-informational?style=for-the-badge" alt="libc: musl"/>
<img src="https://img.shields.io/badge/init-OpenRC-informational?style=for-the-badge" alt="Init: OpenRC"/>
<img src="https://img.shields.io/badge/languages-3-blue?style=for-the-badge" alt="Languages: 3"/>

</div>

---

## 🌐 Select your language

- <img src="assets/flag-spqr.svg" alt="" height="13"> **[Latina](i18n/la/README.md)**
- 🇪🇸 **[Español](i18n/es/README.md)**
- 🇬🇧 **[English](i18n/en/README.md)**

---

## Overview

This project documents one specific advanced Gentoo configuration. It is not a
general Gentoo installation guide. Each decision comes with its reason, and the
guide states what a decision costs as well as what it gives.

The target system uses:

| Component        | Selection                                      |
|------------------|------------------------------------------------|
| Architecture     | AMD64                                          |
| C library        | musl                                           |
| Init system      | OpenRC                                         |
| Encryption       | LUKS2 with Argon2id                            |
| Filesystem       | Btrfs with subvolumes                          |
| Toolchain        | Clang, LLD and libc++                          |
| Optimisation     | ThinLTO                                        |
| Kernel           | Zen                                            |
| Hardening        | The `hardened` profile on top of `musl/llvm`   |
| Logical volumes  | LVM as an optional variant                     |

Gentoo has no profile that combines musl, hardening and LLVM. This guide starts
from the official `musl-llvm-openrc` stage3 and puts the `hardened` features on
top through a stacked profile. [The toolchain
document](i18n/en/toolchain.md) shows the method and its limits.

## Documentation

Each language has the same file tree:

```
i18n/en/                       i18n/es/
├── README.md                  ├── README.md
├── installation.md            ├── installation.md
├── storage.md                 ├── storage.md
├── toolchain.md               ├── toolchain.md
├── optimisation.md            ├── optimisation.md
└── troubleshooting.md         └── troubleshooting.md
```

## Warnings

The `musl/llvm` and `musl/hardened` profiles are experimental in the Gentoo
tree. Software that builds with GCC and glibc does not always build with Clang
and musl.

The partitioning procedures destroy data. Make backups first.

## Contributing

[CONTRIBUTING.md](CONTRIBUTING.md) gives the commit conventions and the rules
that keep the translations synchronised.

## Licence

Except where otherwise noted, this documentation is licensed under the Creative
Commons Attribution-ShareAlike 4.0 International Licence (CC BY-SA 4.0). See
[LICENCE.md](LICENCE.md).

The Gentoo name and logo belong to the Gentoo Foundation. This licence does not
cover them. Their use follows the [Gentoo Name and Logo Usage
Guidelines](https://www.gentoo.org/inside-gentoo/foundation/name-logo-guidelines.html),
and the logo above links to the Gentoo website as those guidelines ask.

This guide is a personal project. It speaks for its author alone, and the
Gentoo Foundation has no part in it.
