<!--
i18n/la/remedia.md
@fraxgut
CC-BY-SA-4.0
Recovery procedures for boot, storage and build failures
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/problemas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/troubleshooting.md)**

# Remedia

</div>

---

Defectus huius institutionis tribus momentis accidunt: in initio, in
appositione receptaculi, et in constructione. Haec pagina eos secundum signum
disponit.

## Index

1. [Ad systema institutum redire](#ad-systema-institutum-redire)
2. [GRUB non apparet](#grub-non-apparet)
3. [GRUB apparet sed initium desinit](#grub-apparet-sed-initium-desinit)
4. [Radix apponitur sed subvolumina absunt](#radix-apponitur-sed-subvolumina-absunt)
5. [Hibernatio non restituit](#hibernatio-non-restituit)
6. [Defectus constructionis](#defectus-constructionis)
7. [Profilum impositum nihil agit](#profilum-impositum-nihil-agit)
8. [Nullum rete post initium](#nullum-rete-post-initium)

## Ad systema institutum redire

Fere omnes reparationes chroot in systema institutum poscunt. Ex medio
institutionis incipe. Deinde appositiones denuo fac:

```bash
cryptsetup open /dev/disk/by-partlabel/gentoosys cryptroot

export OPTS="noatime,compress-force=zstd:5,space_cache=v2,nodiscard"
mount -o ${OPTS},subvol=@ /dev/mapper/cryptroot /mnt/gentoo
mount -o ${OPTS},subvol=@varlog /dev/mapper/cryptroot /mnt/gentoo/var/log
mount /dev/disk/by-partlabel/EFI /mnt/gentoo/boot

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys && mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev && mount --make-rslave /mnt/gentoo/dev

cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
chroot /mnt/gentoo /bin/bash
source /etc/profile
```

Cum varietate LVM, gregem voluminum primum activa:

```bash
vgchange -ay
```

## GRUB non apparet

Firmamentum oneratorem initialem non invenit. Haec ordine inspice:

**Propositum institutionis.** UEFI et BIOS proposita diversa adhibent.
Propositum falsum oneratorem instituit quem firmamentum currere non potest:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-install --target=i386-pc /dev/nvme0n1
```

**Vox firmamenti.** In UEFI, probā vocem exsistere et in ordine initiali esse:

```bash
efibootmgr -v
```

Si vox abest, et firmamentum eam neglegit postquam eam denuo facis, in viam
initialem praedefinitam institue. Omne firmamentum UEFI illam viam accipit:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --removable
```

**Forma ESP.** FAT32 esse debet. Cum systemate plicarum alio firmamentum
partitionem legere non potest.

## GRUB apparet sed initium desinit

Signum causam ostendit.

### Tesseram non poscit

Initramfs non habet quod ad capsam reserandam poscit. Eum denuo fac et contenta
probā:

```bash
dracut --force --kver "$(uname -r)"
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

Si `crypt` abest, `/etc/dracut.conf.d/10-gentoo.conf` examina. Si `crypt` adest
et nucleus tamen deficit, compositio nuclei `Crypt target support` vel
`Device mapper support` non habet. Dracut modulum inclusit, sed nucleus
propositum ei non praebet.

### Tesseram poscit sed radicem non invenit

Lineam mandatorum nuclei quam GRUB fecit examina:

```bash
grep -o 'rd.luks[^ ]*\|root=[^ ]*\|rootflags=[^ ]*' /boot/grub/grub.cfg | sort -u
```

`rd.luks.uuid` UUID **partitionis** LUKS esse debet. `root` UUID **apparatus
decrypti** esse debet. Duo identificatores diversi sunt, et eos confundere
error frequens est:

```bash
blkid -s UUID -o value /dev/disk/by-partlabel/gentoosys   # rd.luks.uuid
blkid -s UUID -o value /dev/mapper/cryptroot              # root
```

Postquam `/etc/default/grub` corrigis, compositionem denuo fac:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Claviatura ad petitionem tesserae non respondet

Initramfs moderatores claviaturae non habet. Eos adde et imaginem denuo fac:

```
add_drivers+=" hid_generic usbhid xhci_hcd "
```

### Discum non invenit

Moderator receptaculi abest. Cum `hostonly="yes"`, dracut tantum modulos
ferramentorum praesentium in tempore construendi includit. Si discus mutatus
est, vel si imaginem in alia machina fecisti, eam denuo construe:

```bash
dracut --force --no-hostonly --kver "$(uname -r)"
```

## Radix apponitur sed subvolumina absunt

`/etc/fstab` examina. Singulae voces Btrfs UUID apparatus decrypti adhibent, et
`subvol=` eas distinguit:

```bash
findmnt --verify --verbose
btrfs subvolume list /
```

Loci appositionis in subvolumine `@` exsistere debent. Index absens
appositionem in initio sine nuntio frustratur:

```bash
mkdir -p /home /.snapshots /var/log /var/tmp /var/swap /var/tmp/portage
```

## Hibernatio non restituit

Locus plicae permutationis mutatur si plicam denuo facis. Eum denuo computa et
lineam mandatorum nuclei corrige:

```bash
btrfs inspect-internal map-swapfile -r /var/swap/swapfile
```

Ille valor **non** est valor quem `filefrag` imprimit. Praeterea `resume` ad
apparatum decryptum spectare debet, non ad partitionem LUKS, et initramfs
modulum `resume` poscit.

## Defectus constructionis

### Defectus in tempore nectendi accidit

LTO discrepantias in tempore nectendi invenit quae antea latebant. LTO pro
fasciculo qui deficit claude, non pro toto systemate:

```bash
echo "categoria/fasciculus no-lto" >> /etc/portage/package.env/no-lto
```

[instrumenta.md](instrumenta.md) definitionem ambitus `no-lto` habet.

### Fasciculus cum Clang non construitur

Non omnis programmatura Clang accipit. Primum defectum in tabula errorum
Gentoo quaere. Ut remedium locale, fasciculum cum GCC construe si eum
institutum habes, vel gradum optimationis demitte:

```bash
cat > /etc/portage/env/gcc-o2 <<'EOF'
CC="gcc"
CXX="g++"
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,--as-needed"
EOF
```

### Constructio omnem memoriam consumit

LTO et Clang multo plus memoriae adhibent quam constructio usitata. Singula
opera parallela illam memoriam multiplicant. Opera parallela minue antequam
gradum optimationis minuis:

```ini
MAKEOPTS="-j4 -l8"
```

`/var/tmp/portage` in tmpfs difficultatem auget, quia index temporarius et
compilator eandem memoriam petunt. Dispositio huius libelli illum indicem in
disco servat, in subvolumine proprio.

### JIT LLVM abest

Profilum `hardened` `-jit -orc` ponit. Si fasciculus ea poscit — Mesa exemplum
usitatum est — ea pro illo solo permitte:

```bash
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

## Profilum impositum nihil agit

Examina quid Portage legat:

```bash
eselect profile show
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

Si vexilla absunt, causa plerumque `layout.conf` est. Syntaxis
`repositorium:via` in plica `parent` formam profili declaratam poscit:

```bash
grep profile-formats /var/db/repos/local/metadata/layout.conf
```

```ini
profile-formats = portage-2
```

`eselect repository create` illam lineam non semper scribit. Eam adde. Deinde
profilum denuo elige.

## Nullum rete post initium

```bash
rc-status                        # curritne NetworkManager?
rc-update show | grep -i network # estne in gradu default?
ip -brief link                   # noscitne nucleus interfaciem?
```

Si interfacies abest, nucleus moderatorem eius non habet. Apparatum et modulum
quem poscit inveni:

```bash
lspci -k | grep -A3 -i net
```

Deinde nexum cum `nmtui` compone.

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **Latina** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/problemas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/troubleshooting.md)**

</div>
