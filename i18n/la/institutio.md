<!--
i18n/la/institutio.md
@guterion
CC-BY-SA-4.0
The installation procedure from the live medium to the first boot
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/instalacion.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/installation.md)**

# Institutio

</div>

---

Haec est ratio integra, a medio institutionis usque ad primum initium
systematis instituti. Dispositio receptaculi in [receptaculum.md](receptaculum.md)
est. Instrumenta compilandi in [instrumenta.md](instrumenta.md) sunt.

## Index

1. [Antequam incipias](#antequam-incipias)
2. [Ambitus institutionis](#ambitus-institutionis)
3. [Discus](#discus)
4. [Valores quos supples](#valores-quos-supples)
5. [Stage3](#stage3)
6. [In chroot intrare](#in-chroot-intrare)
7. [Portage et profilum](#portage-et-profilum)
8. [Loci cum musl](#loci-cum-musl)
9. [Nucleus](#nucleus)
10. [Initramfs](#initramfs)
11. [Onerator initialis](#onerator-initialis)
12. [Rete, usores et ministeria](#rete-usores-et-ministeria)
13. [Primum initium](#primum-initium)
14. [Post institutionem](#post-institutionem)

## Antequam incipias

Hic libellus usum Gentoo praesumit. `emerge`, vexilla USE, profila et nuclei
compositionem manualem scire debes. Partitiones, LUKS et systemata plicarum
huius dispositionis quoque scire debes.

Effectus musl ut bibliothecam C adhibet, OpenRC ut systema initiale, Clang et
LLD ut instrumenta compilandi, et nucleum Zen. Nihil horum praedefinitum apud
Gentoo est. Plura eorum in profilis sunt quae Gentoo experimentalia notat.

Ambitu Linux eges ex quo instituas. Imago officialis Gentoo operatur. Hic
libellus [SystemRescue](https://www.system-rescue.org/) propter instrumenta
adhibet.

Imaginem in apparatum USB ex alia machina Linux scribe:

```bash
lsblk -o NAME,SIZE,MODEL          # apparatum rectum inveni
export USB=/dev/sdX               # X supple
dd if=systemrescue.iso of=${USB} bs=4M status=progress oflag=sync
```

Monitum: hoc mandatum apparatum delet. Litteram inspice antequam id curris.

## Ambitus institutionis

Ex medio incipe. Deinde nexum retis fac:

```bash
nmtui                             # nexum compone, si necesse est
ping -c3 gentoo.org
```

Horologium pone. Tempus falsum probationem subscriptionis et probationem
testimonii frustratur:

```bash
chronyd -q 'pool pool.ntp.org iburst'
```

Per SSH ex alia machina labora. Mandata tunc transcribere potes, et pauciores
errores facis:

```bash
passwd                            # tessera temporaria radicis
systemctl start sshd              # SystemRescue systemd adhibet
ip -brief addr                    # inscriptionem ad quam necteris ostendit
```

## Discus

[receptaculum.md](receptaculum.md) partitiones, capsam LUKS2, subvolumina Btrfs
et plicam permutationis praebet. Illam rationem age antequam pergis.

In fine illius sectionis, systema plicarum radicis in `/mnt/gentoo` esse debet.
`/boot` et cetera subvolumina in loco esse debent:

```bash
findmnt -R /mnt/gentoo
```

## Valores quos supples

Duo genera valorum in mandatis huius libelli apparent.

**Valores qui machinam tuam describunt.** Singuli falsi manent donec eos mutas:

| Valor              | Quid est                                       | Ubi apparet                   |
|--------------------|------------------------------------------------|-------------------------------|
| `/dev/nvme0n1`     | Discus propositus. Nomen falsum discum falsum delet. | `DRIVE`                 |
| `/dev/sdX`         | Apparatus USB medii institutionis              | Gradus medii vivi             |
| `yourhostname`     | Nomen machinae                                 | `/etc/hostname`, `/etc/hosts` |
| `yourusername`     | Nomen tuum ad ingrediendum                     | `useradd`                     |
| `America/Santiago` | Zona horaria tua                               | `/etc/env.d/00musl`           |
| `en_GB.UTF-8`      | Locus tuus                                     | `/etc/env.d/00musl`           |
| `-j16 -l16`        | Numerus filorum processoris, ex `nproc`        | `MAKEOPTS`                    |

**Nomina quae hic libellus eligit.** Sicut sunt operantur. Si unum mutas, id
ubique muta, quia gradus posteriores nomen relegunt:

| Nomen         | Quid nominat                          | Quis relegit                          |
|---------------|---------------------------------------|---------------------------------------|
| `cryptroot`   | Mappa LUKS reserata                   | fstab, linea nuclei, restitutio       |
| `gentoosys`   | Titulus partitionis capsae            | `LUKS_PART_LABEL`, restitutio         |
| `gentoobtrfs` | Titulus systematis plicarum Btrfs     | `BTRFS_LABEL`                         |
| `EFI`, `BOOT` | Tituli partitionum initialium         | Appositiones, fstab, restitutio       |
| `vg_gentoo`   | Grex voluminum varietatis LVM         | Linea mandatorum nuclei               |

Libellus primum genus in variabiles exportat, ideo singulos valores semel
ponis. Iterum eos pone si ex interprete exis et redis.

## Stage3

Hic libellus stage3 `amd64-musl-llvm-openrc` adhibet. Illa plica tres
proprietates figit quas postea sine refectione systematis mutare non potes.
Bibliotheca C est musl. Instrumenta compilandi sunt LLVM. Systema initiale est
OpenRC.

Gentoo indicem constructionis recentissimae divulgat. Ideo diem in mandata non
transcribis:

```bash
cd /mnt/gentoo

export MIRROR="https://distfiles.gentoo.org/releases/amd64/autobuilds"
export STAGE3_PATH=$(curl -s "${MIRROR}/latest-stage3-amd64-musl-llvm-openrc.txt" \
                     | grep -v '^#' | grep 'tar.xz' | cut -d' ' -f1)
export STAGE3_FILE=$(basename "${STAGE3_PATH}")

echo "${STAGE3_FILE}"
```

Plicam, subscriptionem eius et summas eius deprome:

```bash
curl -O "${MIRROR}/${STAGE3_PATH}"
curl -O "${MIRROR}/${STAGE3_PATH}.asc"
curl -O "${MIRROR}/${STAGE3_PATH}.DIGESTS"
```

### Plicam probare

Duae probationes munera diversa habent. Subscriptio ostendit plicam a Gentoo
venire. Summae ostendunt plicam integram advenisse. Utraque tibi opus est.

```bash
# Claves Release Engineering importa
curl -s https://qa-reports.gentoo.org/output/service-keys.gpg | gpg --import

# Subscriptionem tarball probā
gpg --verify "${STAGE3_FILE}.asc" "${STAGE3_FILE}"

# Summam SHA-256 probā
grep -A1 'SHA256' "${STAGE3_FILE}.DIGESTS" | grep "${STAGE3_FILE}$"
sha256sum "${STAGE3_FILE}"
```

`gpg` `Good signature` imprimere debet. Duae summae per singulas litteras
congruere debent. Si una probatio deficit, plicam dele et iterum deprome.

### Plicam expandere

```bash
tar xpvf "${STAGE3_FILE}" --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```

`--xattrs-include` proprietates extensas servat. Facultates programmatum eas
poscunt. `--numeric-owner` prohibet usores ambitus vivi dominium plicarum
mutare.

## In chroot intrare

Compositionem DNS transcribe. Deinde systemata plicarum virtualia appone:

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys  /mnt/gentoo/sys  && mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev  /mnt/gentoo/dev  && mount --make-rslave /mnt/gentoo/dev
mount --bind  /run  /mnt/gentoo/run  && mount --make-slave  /mnt/gentoo/run
```

Intra:

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

> Quotiens ex chroot exis et iterum intras, haec tria mandata age. Variabiles
> quas antea posuisti quoque exporta.

## Portage et profilum

Arborem fasciculorum accipe:

```bash
emerge-webrsync
```

Profilum elige. Hic libellus profilum proprium adhibet, quod proprietates
`hardened` super `musl/llvm` ponit. [instrumenta.md](instrumenta.md) ostendit
quomodo id facias. Interim profilum basale elige:

```bash
eselect profile list | grep musl
eselect profile set default/linux/amd64/23.0/musl/llvm
```

`/etc/portage/make.conf` cum compositione ex [instrumenta.md](instrumenta.md)
scribe. Deinde vexilla processoris et specula adde:

```bash
emerge app-portage/cpuid2cpuflags app-portage/mirrorselect

echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
mirrorselect -i -o >> /etc/portage/make.conf
```

`package.use/00cpu-flags` melius est quam linea `CPU_FLAGS_X86` in
`make.conf`. Portage variabilem per fasciculos administrat. Plicam quoque uno
mandato denuo facis cum ferramenta mutantur.

### Rami fasciculorum

Systema basale in ramo stabili manet. Fasciculi qui tantum in ramo probationis
exsistunt singulas voces accipiunt. `~amd64` pro toto systemate non aperias:

```bash
mkdir -p /etc/portage/package.accept_keywords
echo "sys-kernel/zen-sources ~amd64" >> /etc/portage/package.accept_keywords/kernel
```

Profilum et compositionem systemati instituto applica:

```bash
emerge --ask --verbose --update --deep --newuse @world
```

## Loci cum musl

musl mechanismum locorum glibc non habet. Gentoo `sys-apps/musl-locales` in
arbore principali habet, et in amd64 stabilis est:

```bash
emerge sys-apps/musl-locales sys-libs/timezone-data
```

Zonam horariam et locum in `/etc/env.d` pone:

```bash
cat > /etc/env.d/00musl <<'EOF'
MUSL_LOCPATH="/usr/share/i18n/locales/musl"
CHARSET="UTF-8"
LANG="en_GB.UTF-8"
LC_COLLATE="C"
TZ="America/Santiago"
EOF

env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
date
```

`LC_COLLATE="C"` per punctum codicis ordinat, non per regulas linguae.
Celerius est, et ordinem in scriptis praevidibilem facit. Id muta si ordinem
linguae tuae poscis.

## Nucleus

Firmamentum, fontes et instrumenta initialia institue:

```bash
emerge sys-kernel/linux-firmware sys-kernel/zen-sources
emerge sys-kernel/dracut
```

`sys-kernel/installkernel` opus post constructionem agit. Initramfs facit et
oneratorem initialem renovat quotiens nucleum instituis.

```bash
echo "sys-kernel/installkernel dracut grub" \
    >> /etc/portage/package.use/installkernel
emerge sys-kernel/installkernel
```

Fontes elige:

```bash
eselect kernel list
eselect kernel set 1
cd /usr/src/linux
```

### Compositio

Compositio ex ferramentis tuis pendet. Ea cum `lspci -k`, `lsusb` et `lscpu`
examina. Deinde quod poscis permitte. Hae optiones necessariae sunt ut systema
cum hac dispositione receptaculi incipiat:

| Area                  | Optio                                                     |
|-----------------------|-----------------------------------------------------------|
| Device Drivers        | `Device mapper support` et `Crypt target support`          |
| File systems          | `Btrfs filesystem support`                                |
| File systems          | `VFAT` pro ESP, vel `ext4` pro `/boot` in BIOS            |
| Cryptographic API     | `AES`, `XTS`, `SHA512`                                    |
| General setup         | Auxilium initramfs et compressio quam dracut adhibet      |
| Receptaculum          | Moderator discorum tuorum: NVMe, AHCI vel SCSI            |

Sine `Crypt target support` initramfs capsam reserare non potest. Sine
moderatore disci discum invenire non potest. Hae duae causae saepissime primum
initium frustrantur.

```bash
make menuconfig
```

### Constructio

Clang, LLD et assemblator inclusus nucleum construunt:

```bash
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

Ultimum mandatum `installkernel` vocat. Illud instrumentum initramfs facit et
compositionem GRUB denuo scribit.

## Initramfs

Dracut compone antequam nucleum instituis. `installkernel` tunc initramfs
plenum facit:

```bash
cat > /etc/dracut.conf.d/10-gentoo.conf <<'EOF'
# Builds an initramfs for this machine alone. The image stays small
# because it holds the modules this hardware needs and nothing else.
hostonly="yes"

# Zstd needs matching support in the kernel.
compress="zstd"

# crypt opens the LUKS2 container, btrfs mounts the root subvolume and
# resume restores a hibernated image.
add_dracutmodules+=" crypt btrfs resume "

# Keyboard drivers, so the passphrase can be typed on a USB keyboard.
add_drivers+=" hid_generic usbhid xhci_hcd "
EOF
```

Si varietate LVM uteris, `lvm` ad `add_dracutmodules` adde.

Effectum probā:

```bash
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

## Onerator initialis

GRUB 2.14 auxilium Argon2 in ipso incepto fert, et in amd64 stabilis est.
Id emerge, et cum capsa LUKS2 sicut est operatur.

`/boot` extra capsam encryptam manet. GRUB nucleum et initramfs apertos legit
et eos incipit. Initramfs deinde LUKS reserat. Systema ergo tesseram semel
poscit, post indicem GRUB:

```
firmamentum → GRUB → nucleus + initramfs → tessera → radix Btrfs → OpenRC
```

Dispositio quae etiam `/boot` encryptat tesseram bis poscit. GRUB primum
poscit, ut `/boot` legat, et initramfs iterum poscit. Duo programmata in
ambitibus separatis currunt, et GRUB nucleo sessionem reseratam non tradit.
Utraque petitio eandem tesseram accipit et eundem keyslot reserat. Hic libellus
unam petitionem servat, et `GRUB_ENABLE_CRYPTODISK` extra compositionem manet.

```bash
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf   # vel "pc" in BIOS
emerge sys-boot/grub
```

Identificatores pro linea mandatorum nuclei collige:

```bash
export LUKS_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${LUKS_PART_LABEL})
export RESUME_OFFSET=$(btrfs inspect-internal map-swapfile -r /var/swap/swapfile)
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
```

`/etc/default/grub` compone:

```ini
# rd.luks.uuid           capsam quam initramfs reserat significat
# rd.luks.allow-discards TRIM per dm-crypt permittit (profilum B)
# root, rootflags        apparatus radicis et subvolumen quod in eo apponitur
# resume, resume_offset  hibernatio in plicam permutationis
GRUB_CMDLINE_LINUX="rd.luks.uuid=${LUKS_UUID} rd.luks.allow-discards root=UUID=${ROOT_UUID} rootflags=subvol=@ resume=UUID=${ROOT_UUID} resume_offset=${RESUME_OFFSET}"
GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3"
```

GRUB institue.

Pro UEFI:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
```

Pro BIOS:

```bash
grub-install --target=i386-pc $DRIVE
```

Deinde compositionem fac:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Exitum lege. GRUB nucleum et initramfs eius invenire debet. Vox indicis
parametra supra scripta continere debet.

## Rete, usores et ministeria

### Nomen machinae

Nomen machinae in singulis retibus quibus se iungit machinam significat, ideo
in illis retibus unicum esse debet. Duae machinae eodem nomine in eadem rete
locali confliguntur, et mDNS secundam in `nomen-2` renominat. Nomen unicum in
orbe terrarum esse non debet.

RFC 1123 syntaxim praebet:

- Ab 1 ad 63 litteras.
- Litterae, numeri et lineola. Numerus nomen incipere potest.
- Lineola nomen numquam incipit neque finit.
- Nulla puncta et nullae lineolae infimae. Punctum partes nominis dominii dividit.

Litterae minusculae mos sunt, quia DNS inter maiusculas et minusculas non
distinguit.

Ministerium `hostname` systematis OpenRC primum `/etc/hostname` legit.
Variabilem `hostname` in `/etc/conf.d/hostname` adhibet cum illa plica vacua
est.

```bash
printf '%s\n' "yourhostname" > /etc/hostname
```

OpenRC idem nomen in `/etc/hosts` poscit. Illa vox machinae permittit nomen
suum per inquisitionem localem invenire:

```
127.0.0.1   yourhostname.localdomain yourhostname localhost
::1         yourhostname.localdomain yourhostname localhost
```

### Usores

```bash
passwd                                   # tessera perpetua radicis

useradd -m -G wheel,users,audio,video,portage yourusername
passwd yourusername
```

Grex `wheel` privilegia radicis per `doas` accipit. Grex `portage` rationi
permittit indices constructionis legere.

Moderatorem retis et ministeria systematis institue:

```bash
emerge net-misc/networkmanager app-admin/metalog net-misc/chrony
emerge app-admin/doas sys-process/cronie

rc-update add NetworkManager default
rc-update add metalog default
rc-update add chronyd default
rc-update add cronie default
```

`cronie` purgationem hebdomadalem ex [receptaculum.md](receptaculum.md) agit.

Gregi `wheel` privilegia radicis permitte:

```bash
echo "permit persist :wheel" > /etc/doas.conf
chmod 0400 /etc/doas.conf
```

Ministerio `dmcrypt` tantum pro pluribus voluminibus encryptis eges quae post
initium reserantur. Initramfs apparatum radicis reserat.

## Primum initium

Ex chroot exi. Deinde ordine contrario depone:

```bash
exit
cd /
swapoff /mnt/gentoo/var/swap/swapfile
umount -R /mnt/gentoo
cryptsetup close cryptroot
reboot
```

Medium institutionis tolle. GRUB apparere debet. Initramfs tesseram LUKS
poscere debet antequam systema plicarum radicis apponit.

Si quid deficit, [remedia.md](remedia.md) causas usitatas praebet.

## Post institutionem

Systema nunc cum musl, OpenRC, Clang et nucleo Zen incipit. Duo opera manent:
profilum impositum cum proprietatibus munitionis, et refectio systematis cum
LTO. [instrumenta.md](instrumenta.md) utrumque ostendit.

Illud opus in systemate instituto age, non in chroot. Si refectio quid frangit,
systema quod incipit habes, et ex eo id reficere potes.

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/instalacion.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/installation.md)**

</div>
