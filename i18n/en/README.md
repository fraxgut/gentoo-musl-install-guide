<!--
i18n/en/README.md
@fraxgut
CC-BY-SA-4.0
English landing page and index for the installation guide
-->

# Advanced Gentoo Linux installation guide

> 🌐 **Language:** [Latina](../la/README.md) · [Español](../es/README.md) · **English**

A Gentoo Linux installation on AMD64 with musl, OpenRC, LUKS2, Btrfs,
LLVM/Clang, LTO and the Zen kernel.

## About this guide

This guide documents one specific configuration. It is not a general Gentoo
installation. Each part follows an explicit decision, and the text gives the
reason together with the procedure.

The guide is for people with experience in Gentoo. It assumes that you know
`emerge`, USE flags, profiles and manual kernel configuration.

## Target configuration

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

## Documentation

1. [Installation](installation.md) — from the installation medium to the first boot.
2. [Storage layout](storage.md) — partitioning, LUKS2, Btrfs, compression, TRIM and swap.
3. [LLVM toolchain and LTO](toolchain.md) — the stacked profile, the hardening and the optimisation.
4. [Optimisation](optimisation.md) — the tiers, the hardware gates and the escape hatches.
5. [Troubleshooting](troubleshooting.md) — recovery by symptom.

## Warnings

The Gentoo tree marks the `musl/llvm` and `musl/hardened` profiles as
experimental. Software that builds with GCC and glibc does not always build
with Clang and musl.

The C library, the toolchain and the init system become fixed when you unpack
the stage3. To change them later, you must rebuild the system. You thus make
the decision before the installation.

The partitioning procedures destroy data. Make backups. Check the device name
two times.

## Roadmap

- [x] Btrfs on LUKS2 with subvolumes
- [x] musl with OpenRC
- [x] Clang, LLD and libc++ from the official stage3
- [x] Hardening stacked on the LLVM profile
- [x] ThinLTO with the Gentoo support
- [x] Zen kernel built with LLVM
- [x] Native Btrfs swap with hibernation
- [ ] Secure Boot, to authenticate the boot chain that stays outside LUKS
- [ ] SELinux
- [ ] An example kernel configuration
- [ ] Automatic snapshots and boot from a snapshot

## Licence

Except where otherwise noted, this documentation is licensed under the Creative
Commons Attribution-ShareAlike 4.0 International Licence (CC BY-SA 4.0). See
[LICENCE.md](../../LICENCE.md).

The Gentoo name and logo belong to the Gentoo Foundation. This licence does not
cover them. Their use follows the [Gentoo Name and Logo Usage
Guidelines](https://www.gentoo.org/inside-gentoo/foundation/name-logo-guidelines.html).

This guide is a personal project. It speaks for its author alone, and the
Gentoo Foundation has no part in it.

## Contributing

Contributions are welcome. [CONTRIBUTING.md](../../CONTRIBUTING.md) gives the
commit conventions and the rules to keep the translations synchronised.

## Contact

Franco Gutiérrez — [@fraxgut](https://github.com/fraxgut) — franco.gutierrez.1@ug.uchile.cl

## References

- [Gentoo AMD64 Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Gentoo musl project](https://wiki.gentoo.org/wiki/Project:Musl)
- [Portage custom profiles](https://wiki.gentoo.org/wiki/Portage/Profiles/Custom_profiles)
- [LLVM on Gentoo](https://wiki.gentoo.org/wiki/LLVM)
- [LTO on Gentoo](https://wiki.gentoo.org/wiki/LTO)
- [Btrfs documentation](https://btrfs.readthedocs.io/)
- [dm-crypt full disk encryption](https://wiki.gentoo.org/wiki/Dm-crypt_full_disk_encryption)
- [Dracut on Gentoo](https://wiki.gentoo.org/wiki/Dracut)

---

> 🌐 **Language:** [Latina](../la/README.md) · [Español](../es/README.md) · **English**
