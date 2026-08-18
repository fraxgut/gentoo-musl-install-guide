<!--
i18n/la/receptaculum.md
@fraxgut
CC-BY-SA-4.0
Storage layout: partitioning, LUKS2, Btrfs subvolumes and the swapfile
-->

# Dispositio receptaculi

> 🌐 **Lingua:** **Latina** · [Español](../es/almacenamiento.md) · [English](../en/storage.md)

Hoc documentum dispositionem receptaculi huius libelli ostendit. Causam
cuiusque consilii quoque praebet. Ratio institutionis integra in
[institutio.md](institutio.md) invenitur.

## Index

1. [Strata](#strata)
2. [Praeparatio disci](#praeparatio-disci)
3. [Partitiones](#partitiones)
4. [Encryptio LUKS2](#encryptio-luks2)
5. [Btrfs et subvolumina](#btrfs-et-subvolumina)
6. [Optiones apponendi](#optiones-apponendi)
7. [Compressio](#compressio)
8. [TRIM et discard](#trim-et-discard)
9. [Systema apponere](#systema-apponere)
10. [Spatium permutationis](#spatium-permutationis)
11. [fstab](#fstab)
12. [Reserare in initio](#reserare-in-initio)
13. [Varietas cum LVM](#varietas-cum-lvm)

## Strata

Institutio praedefinita tribus stratis utitur:

```
Discus corporeus
    │
    ├── GPT
    │
    ├── partitio 1 ── ESP (FAT32)  vel  BIOS boot + /boot
    │
    └── partitio 2 ── LUKS2 (Argon2id)
                          │
                          └── Btrfs
                                 ├── @           →  /
                                 ├── @home       →  /home
                                 ├── @snapshots  →  /.snapshots
                                 ├── @varlog     →  /var/log
                                 ├── @vartmp     →  /var/tmp
                                 ├── @portage    →  /var/tmp/portage
                                 └── @swap       →  /var/swap
```

`/boot` extra capsam encryptam manet. Hoc consilium effectum magni momenti
habet. GRUB nucleum et initramfs ex partitione aperta legit, ideoque GRUB LUKS
non reserat. Argon2id semel currit, intra initramfs, ubi `cryptsetup` memoriam
et tempus processoris habet quae munus poscit.

Systema ergo tesseram semel poscit, post indicem GRUB. Dispositio quae etiam
`/boot` encryptat `GRUB_ENABLE_CRYPTODISK` ponit, et bis poscit: GRUB poscit ut
`/boot` legat, et initramfs iterum poscit propter apparatum radicis. Hic
libellus unam petitionem servat.

### Catena initialis

LUKS omnibus quae in capsa sunt secretum praebet. GRUB, nucleus et initramfs
extra eam manent, quia machina ea legere debet antequam tesseram habet.

Hoc duas proprietates securitatis dividit. Qui discum aufert nulla data tua
legit. Qui accessum corporeum saepe habet nucleum vel initramfs mutare potest,
et initramfs mutatus tesseram in initio proximo capit.

Encryptio ipsius `/boot` huic tantum ex parte respondet, quia GRUB ipse legi et
mutari potest. Responsum plenum subscriptionem eorum quae machina incipit
probat:

```
Secure Boot  →  probatio subscriptionis  →  GRUB, nucleus, initramfs
                                                  ↓
                                            LUKS2 + Argon2id
```

Cum partes initiales subscriptae sunt, libellus `/boot` apertum servare potest
et unam tesserae petitionem retinere. Hoc opus in itinere proposito est.

### Cur Btrfs recta super LUKS2 ponitur

Btrfs iam unum receptaculum praebet cum subvolumnibus, imaginibus momentaneis,
partibus definitis et pluribus apparatibus. Grex voluminum LVM cuius unum
volumen logicum centesimas centum spatii implet stratum addit quod nihil
administrat. Btrfs singulas amplificationes, singulas imagines et singulas
spatii reservationes ipse agit.

LVM utile fit cum dispositio plura volumina independentia in eadem capsa
encrypta poscit. Exemplum est unum volumen machinis virtualibus, aliud cum
systemate plicarum diverso, et spatium in grege non assignatum. Sectio
[Varietas cum LVM](#varietas-cum-lvm) eam dispositionem ostendit.

## Praeparatio disci

Discum propositum inveni. Nomen apparatus bis inspice. Mandata huius sectionis
omnia data illius disci delent.

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

Variabilem ad discum quem elegisti pone:

```bash
export DRIVE=/dev/nvme0n1
```

Data fortuita in discum scribe antequam capsam facis. Adversarius tunc discernere
non potest quae partes data encrypta teneant, et quae numquam scriptae sint.
Hic gradus optionalis est. Tantum temporis poscit quantum una scriptio disci
plena.

```bash
cryptsetup open --type plain --key-file /dev/urandom $DRIVE purgatio
dd if=/dev/zero of=/dev/mapper/purgatio status=progress bs=16M
cryptsetup close purgatio
```

Nullae per mappam encryptam simplicem transeuntes fiunt data fortuitis
similia. Haec ratio multo celerior est quam lectio ex `/dev/urandom`.

Deinde omnem tabulam partitionum veterem dele:

```bash
sgdisk --zap-all $DRIVE
```

## Partitiones

Primum inveni utrum systema per UEFI an per BIOS incipiat:

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

**Unam** ex duabus dispositionibus elige.

### UEFI

Una partitio systematis EFI et capsa encrypta:

```bash
export DISK_LABEL="gentoosys"

sgdisk --clear \
       --new=1:0:+1GiB --typecode=1:ef00 --change-name=1:EFI \
       --new=2:0:0     --typecode=2:8309 --change-name=2:${DISK_LABEL} \
       $DRIVE

export EFI_PART_LABEL=EFI
export LUKS_PART_LABEL=${DISK_LABEL}
```

Codex generis `8309` partitionem LUKS significat. Codex `8300` quoque
operatur. Sed `8309` contentum recte describit, et instrumenta systematis tunc
sciunt partitionem data encrypta tenere.

Unum gibibyte pro ESP plures nucleos cum suis imaginibus initramfs capit. Hoc
etiam imagines comprehendit quas inter probationes initiales facis.

### BIOS

Tres partitiones: area pro secundo gradu GRUB, `/boot` separatum, et capsa
encrypta.

```bash
export DISK_LABEL="gentoosys"

sgdisk --clear \
       --new=1:0:+1MiB  --typecode=1:ef02 --change-name=1:GRUB \
       --new=2:0:+1GiB  --typecode=2:8300 --change-name=2:BOOT \
       --new=3:0:0      --typecode=3:8309 --change-name=3:${DISK_LABEL} \
       $DRIVE

export BOOT_PART_LABEL=BOOT
export LUKS_PART_LABEL=${DISK_LABEL}
```

## Encryptio LUKS2

`cryptsetup` Argon2id ut munus praedefinitum clavis derivandae adhibet a
versione 2.4.0. RFC 9106 Argon2id variantem principalem facit, quia duas
defensiones coniungit. Ab Argon2i resistentiam contra impetus per canales
laterales accipit. Ab Argon2d resistentiam contra permutationes inter tempus et
memoriam.

```bash
cryptsetup luksFormat \
    --type luks2 \
    --cipher aes-xts-plain64 \
    --key-size 512 \
    --hash sha512 \
    --pbkdf argon2id \
    --iter-time 5000 \
    --use-random \
    --verify-passphrase \
    /dev/disk/by-partlabel/${LUKS_PART_LABEL}
```

`--iter-time 5000` quinque secunda derivationi clavis dat. Cum eo sumptu
`cryptsetup` memoriam et iterationes quas machina fert calibrat. Pretium
impetus per vim brutam ergo ferramenta tua sequitur, non constantem fixam.

Tessera cetera omnia custodit. Elige tesseram quae impetui per lexicon
resistat. Plura verba fortuita meliora sunt quam catena brevis cum litteris
mutatis.

Capsam reserā:

```bash
cryptsetup open /dev/disk/by-partlabel/${LUKS_PART_LABEL} cryptroot
```

Apparatus decryptus est `/dev/mapper/cryptroot`.

> Si sessio desinit et iterum incipis, capsam hoc mandato denuo reserā.

## Btrfs et subvolumina

```bash
export BTRFS_LABEL="gentoobtrfs"
mkfs.btrfs --label ${BTRFS_LABEL} /dev/mapper/cryptroot
```

Summum gradum systematis plicarum appone ut subvolumina facias:

```bash
mkdir -p /mnt/gentoo
mount /dev/mapper/cryptroot /mnt/gentoo

btrfs subvolume create /mnt/gentoo/@
btrfs subvolume create /mnt/gentoo/@home
btrfs subvolume create /mnt/gentoo/@snapshots
btrfs subvolume create /mnt/gentoo/@varlog
btrfs subvolume create /mnt/gentoo/@vartmp
btrfs subvolume create /mnt/gentoo/@portage
btrfs subvolume create /mnt/gentoo/@swap

umount /mnt/gentoo
```

Singula subvolumina propositum habent:

| Subvolumen   | Locus appositionis | Propositum                                  |
|--------------|--------------------|---------------------------------------------|
| `@`          | `/`                | Radix systematis instituti.                 |
| `@home`      | `/home`            | Data usorum per novam institutionem servat.  |
| `@snapshots` | `/.snapshots`      | Imagines momentaneas tenet.                 |
| `@varlog`    | `/var/log`         | Acta extra imagines momentaneas servat.     |
| `@vartmp`    | `/var/tmp`         | Plicas temporarias secernit.                |
| `@portage`   | `/var/tmp/portage` | Optiones appositionis proprias accipit.     |
| `@swap`      | `/var/swap`        | Plicam permutationis tenet.                 |

`@portage` explicationem poscit. Portage decem gibibytes et amplius inter
constructionem magnam scribit et delet, et illae plicae post paucas minutas
abeunt. Subvolumen separatum permittit ut cum compressione vili apponatur, vel
sine compressione, dum cetera subvolumina compositionem acrem servant.

## Optiones apponendi

Compositio praedefinita huius libelli:

```
noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@
```

Et pro `@portage`:

```
noatime,compress=zstd:1,space_cache=v2,nodiscard,subvol=@portage
```

Tres notae de eo quomodo libellus illam seriem elegit:

**Btrfs genus apparatus ipse invenit.** Topographiam apparatus legit et invenit
utrum apparatus rotans sit. Libellus id opus Btrfs relinquit. `ssd` manu tantum
pone si stratum virtuale proprietatem falso ostendit. Optimationes veteres
dispositionis pro SSD abierunt, quia in ferramentis hodiernis nihil praebebant
et fragmentationem augere poterant.

**Libellus optiones quas vult scribit.** `defaults` optiones iam activas
colligit, ideo series brevis manet et singulae voces significationem ferunt.

**`space_cache=v2` explicite ponitur,** quamquam nunc mos praedefinitus est.
Libellus compositionem propositam declarat, et illa declaratio stabilis manet
dum valores praedefiniti nuclei mutantur. Unum pretium est congruentia:
scriptiones nucleum cum auxilio arboris spatii liberi poscunt, quem omnis
nucleus hodiernus habet.

## Compressio

`compress-force` Btrfs iubet compressionem in singulis extensionibus temptare.
Si effectus compressus maior est quam ingressus, Btrfs data sine compressione
scribit. Ideo nulla plica maior fit. Pretium est tempus processoris in temptatione
irrita.

Documenta Btrfs `compress` simplicem pro usu communi commendant, quia
heuristicae hodiernae bene iudicant. Hic libellus `compress-force` duabus de
causis eligit. Prima causa est amplitudo: plica potest incipere a datis
incompressibilibus et pergere cum datis valde compressibilibus, et sine `force`
consilium maturum omnia sequentia excludere potest. Secunda causa est
praelatio consilii explicti supra heuristicam quae inter versiones nuclei
mutatur.

### Gradum eligere

Btrfs gradus zstd in tres partes disponit. Gradus 1 ad 3 fere in tempore reali
operantur. Gradus 4 ad 8 tardiores sunt et melius comprimunt. Gradus 9 ad 15
multo plus laboris poscunt, et lucrum magnitudinis parvum esse potest.

Una proprietas inaequalis electionem decernit. **Pretium comprimendi cum gradu
crescit. Pretium decomprimendi fere constans manet.** Gradus alti ergo
scriptiones tardant et lectiones fere intactas relinquunt.

| Gradus    | Munus in libello   | Quando convenit                                                       |
|-----------|--------------------|-----------------------------------------------------------------------|
| `zstd:5`  | Praedefinitus      | Statio laboris hodierna: aequilibrium bonum inter spatium et processorem. |
| `zstd:9`  | Compressio alta    | Processor abundans, et receptaculum pretiosius quam tempus scribendi. |
| `zstd:15` | Compressio maxima  | Data stabilia quae semel scribis et saepe legis.                      |

`zstd:5` valor praedefinitus huius libelli est. Plus comprimit quam valor
praedefinitus Btrfs et efficientiam scribendi rationabilem in ferramentis
hodiernis servat.

`zstd:9` et `zstd:15` optiones plane validae sunt. Onus laboris decernit, non
celeritas machinae. Processor cum multis nucleis `zstd:15` rectum pro
`/var/tmp/portage` non facit. Ibi processorem in plicis consumis quas post
paucas minutas deles, in certamine cum constructione. Pro copia datorum
stabili quae menses in disco manet, effectus alius est.

Hac de causa `@portage` `compress=zstd:1` adhibet. Compressio vilissima est,
sine `force`, ut Portage et Clang nucleos habeant.

## TRIM et discard

TRIM apparatui nuntiat qui frusta data utilia non iam teneant. Sine ea
notitia, moderator SSD operatur quasi multo minus spatii liberi habeat quam
revera habet. Hoc collectionem purgamentorum et efficientiam diuturnam
deteriorat.

Pretium est secretum. Documenta nuclei dicunt discard per dm-crypt notitiam de
volumine ostendere: quae regiones liberae sint, quantum spatii adhibeas, et
quasdam proprietates systematis plicarum. Clavis, nomina plicarum et contenta
plicarum tuta manent. Adversarius formam assignationis videt.

TRIM per omnia strata transire debet. Ideo consilium duobus locis capis: in
Btrfs et in dm-crypt.

| Profilum          | dm-crypt         | Btrfs            | Praefert                       |
|-------------------|------------------|------------------|--------------------------------|
| **A** Secretum maximum | sine discard | `nodiscard`      | Obscuritatem formae assignationis |
| **B** Aequilibratum | `allow-discards` | `nodiscard` + `fstrim` hebdomadale | Secretum temporis et salutem SSD |
| **C** Efficientia | `allow-discards` | `discard=async`  | Constantiam diuturnam SSD      |

**Hic libellus profilo B utitur.**

Profilum A obscuritatem maximam servat. Sed omnem notitiam de spatio libero in
SSD dimittit. In machina Gentoo pretium cum tempore crescit. Singulae
constructiones magnae decem gibibytes et amplius scribunt et abiciunt, et
moderator tunc discum administrat quem multo pleniorem credit quam est.

Profilum C apparatui imaginem fere continuam spatii liberati dat. Btrfs
extensiones colligit et operationes moderatur, ideo effectus in efficientiam
parvus est. Vicissim adversarius cum accessu ad receptaculum notitiam recentem
accipit de eo quod liberum factum est, et quando.

Profilum B TRIM per greges agit dum machina otiosa est. Fere omne commodum
practicum profili C accipit. Pagina manualis `fstrim` unam executionem
hebdomadalem satis esse dicit pro scrinio vel servitore usitato. Profilum B
etiam resolutionem temporis tollit. Singulas liberationes eo momento quo fiunt
non nuntiat. Unum gregem magnum semel in hebdomada mittit. Forma spatii liberi
tandem apparet. Linea temporis est quod secretum servas.

`allow-discards` in linea mandatorum nuclei ponitur, cum ceteris parametris
dracut:

```
rd.luks.allow-discards
```

Deinde purgationem periodicam cum cron dispone:

```bash
emerge sys-process/cronie
rc-update add cronie default

cat > /etc/cron.weekly/fstrim <<'EOF'
#!/bin/sh
# Releases unused blocks to the device one time each week. The batch runs
# while the machine is idle. The discard requests thus stay large, and
# they do not interfere with interactive work.
exec /usr/sbin/fstrim --all --quiet
EOF
chmod +x /etc/cron.weekly/fstrim
```

Ut profilo C utaris, `nodiscard` cum `discard=async` in optionibus appositionis
commuta et opus cron omitte. Ut profilo A utaris, `rd.luks.allow-discards` ex
linea mandatorum nuclei tolle.

## Systema apponere

```bash
export BTRFS_OPTS="noatime,compress-force=zstd:5,space_cache=v2,nodiscard"
export PORTAGE_OPTS="noatime,compress=zstd:1,space_cache=v2,nodiscard"

mount -o ${BTRFS_OPTS},subvol=@ /dev/mapper/cryptroot /mnt/gentoo

mkdir -p /mnt/gentoo/{boot,home,.snapshots,var/log,var/tmp,var/swap}

mount -o ${BTRFS_OPTS},subvol=@home      /dev/mapper/cryptroot /mnt/gentoo/home
mount -o ${BTRFS_OPTS},subvol=@snapshots /dev/mapper/cryptroot /mnt/gentoo/.snapshots
mount -o ${BTRFS_OPTS},subvol=@varlog    /dev/mapper/cryptroot /mnt/gentoo/var/log
mount -o ${BTRFS_OPTS},subvol=@vartmp    /dev/mapper/cryptroot /mnt/gentoo/var/tmp
mount -o ${BTRFS_OPTS},subvol=@swap      /dev/mapper/cryptroot /mnt/gentoo/var/swap

mkdir -p /mnt/gentoo/var/tmp/portage
mount -o ${PORTAGE_OPTS},subvol=@portage /dev/mapper/cryptroot /mnt/gentoo/var/tmp/portage

chmod 1777 /mnt/gentoo/var/tmp
chmod 775  /mnt/gentoo/var/tmp/portage
```

Deinde partitionem initialem appone.

Pro UEFI:

```bash
mkfs.vfat -F 32 -n EFI /dev/disk/by-partlabel/${EFI_PART_LABEL}
mount -o rw,nosuid,nodev,noexec,relatime,fmask=0077,dmask=0077 \
      /dev/disk/by-partlabel/${EFI_PART_LABEL} /mnt/gentoo/boot
```

Larvae `0077` partitionem ad `root` restringunt. ESP permissiones POSIX non
sustinet, ideo restrictionem in appositione ponis. Sine larvis omnis usor
nucleum et initramfs legere potest.

Pro BIOS:

```bash
mkfs.ext4 -L BOOT /dev/disk/by-partlabel/${BOOT_PART_LABEL}
mount -o rw,nosuid,nodev,relatime \
      /dev/disk/by-partlabel/${BOOT_PART_LABEL} /mnt/gentoo/boot
```

## Spatium permutationis

A versione 6.1 `btrfs-progs`, Btrfs mandatum habet quod plicam permutationis
cum proprietatibus rectis facit. Plica nullam copiam in scribendo et nullam
compressionem accipit. Mandatum etiam `mkswap` in plica exsequitur.

```bash
btrfs filesystem mkswapfile --size 8G /mnt/gentoo/var/swap/swapfile
swapon /mnt/gentoo/var/swap/swapfile
```

Illud unum mandatum seriem `truncate`, `chattr +C`, `dd` et `mkswap` supplet.

Hac tabula pro magnitudine utere:

| Memoria | Sine hibernatione | Cum hibernatione |
|---------|-------------------|------------------|
| 4 GiB   | 2 GiB             | 6 GiB            |
| 8 GiB   | 3 GiB             | 11 GiB           |
| 16 GiB  | 4 GiB             | 20 GiB           |
| 32 GiB  | 6 GiB             | 38 GiB           |
| 64 GiB  | 8 GiB             | 72 GiB           |

### Hibernatio

Hibernatio locum corporeum plicae in systemate plicarum poscit. Ille valor
**non** est valor quem `filefrag` imprimit. `btrfs-progs` eum computat:

```bash
btrfs inspect-internal map-swapfile -r /mnt/gentoo/var/swap/swapfile
```

Illum numerum nucleo ut `resume_offset` da. `resume=` quoque ad apparatum
decryptum pone. [institutio.md](institutio.md) compositionem plenam ostendit.

## fstab

Plicam permutationis **per viam** nomina. Numerus UUID quem `mkswap` scribit
systema permutationis significat. Intra plicam manet, ideo neque visibilis
neque utilis est ut identificator apparatus. Vox `UUID=` pro plica
permutationis non operatur.

```bash
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
export BOOT_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${EFI_PART_LABEL})
```

```
# <systema plicarum>     <locus appositionis> <genus> <optiones>                                                     <dump> <pass>

UUID=${ROOT_UUID}        /                   btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@            0 0
UUID=${ROOT_UUID}        /home               btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@home        0 0
UUID=${ROOT_UUID}        /.snapshots         btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@snapshots   0 0
UUID=${ROOT_UUID}        /var/log            btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@varlog      0 0
UUID=${ROOT_UUID}        /var/tmp            btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@vartmp      0 0
UUID=${ROOT_UUID}        /var/swap           btrfs   noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@swap        0 0
UUID=${ROOT_UUID}        /var/tmp/portage    btrfs   noatime,compress=zstd:1,space_cache=v2,nodiscard,subvol=@portage           0 0
UUID=${BOOT_UUID}        /boot               vfat    rw,nosuid,nodev,noexec,relatime,fmask=0077,dmask=0077                      0 2
/var/swap/swapfile       none                swap    defaults                                                                   0 0
```

Columna sexta `0` est in singulis vocibus Btrfs. `fsck` in Btrfs nihil agit.
Btrfs summas suas probat cum data legit.

## Reserare in initio

Initramfs capsam LUKS reserat antequam systema plicarum radicis apponit.
Parametra nuclei ex dracut hoc moderantur. Cum ea compositione, ministerium
`dmcrypt` systematis OpenRC in apparatu radicis nihil agit.

`dmcrypt` tantum pro pluribus voluminibus encryptis quae post initium
reserantur necessarium est. In OpenRC id in `/etc/conf.d/dmcrypt` componis.
Syntaxis a syntaxi `/etc/crypttab` differt:

```bash
# /etc/conf.d/dmcrypt
target=data
source='UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
```

```bash
rc-update add dmcrypt boot
```

## Varietas cum LVM

LVM utile fit cum capsa encrypta plura volumina independentia tenere debet.
Gregem voluminum in apparatu decrypto facis:

```
GPT
 └── LUKS2
       └── LVM2 (grex voluminum)
             ├── lv_root  → Btrfs   →  /
             ├── lv_vm    → XFS     →  imagines machinarum virtualium
             └── spatium liberum reservatum
```

```bash
pvcreate /dev/mapper/cryptroot
vgcreate vg_gentoo /dev/mapper/cryptroot

lvcreate --size 200G --name lv_root vg_gentoo
lvcreate --size 300G --name lv_vm   vg_gentoo

mkfs.btrfs --label gentoobtrfs /dev/vg_gentoo/lv_root
mkfs.xfs -L vmstore /dev/vg_gentoo/lv_vm
```

Spatium aliquod in grege non assignatum relinque si volumen postea amplificare
cogitas. Illa flexibilitas est quam LVM praebet et Btrfs non praebet.

Cum hac varietate, systema plicarum radicis ex `/dev/mapper/cryptroot` ad
`/dev/vg_gentoo/lv_root` migrat. Appositiones, `fstab` et lineam mandatorum
nuclei muta. Linea mandatorum nuclei hoc parametrum quoque poscit:

```
rd.lvm.lv=vg_gentoo/lv_root
```

Initramfs modulum `lvm` includere debet. Ministerium `lvm` systematis OpenRC in
gradu `boot` esse debet:

```bash
rc-update add lvm boot
```

Si dispositio tua unum volumen logicum habet quod gregem implet, LVM nihil
administrat. Dispositione praedefinita huius libelli utere.

---

> 🌐 **Lingua:** **Latina** · [Español](../es/almacenamiento.md) · [English](../en/storage.md)
