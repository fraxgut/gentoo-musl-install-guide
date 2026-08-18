<!--
i18n/en/storage.md
@fraxgut
CC-BY-SA-4.0
Storage layout: partitioning, LUKS2, Btrfs subvolumes and the swapfile
-->

# Storage layout

> 🌐 **Language:** [Latina](../la/receptaculum.md) · [Español](../es/almacenamiento.md) · **English**

This document shows the storage design of the guide. It also gives the reason
for each decision. The full installation procedure is in
[installation.md](installation.md).

## Contents

1. [The layers](#the-layers)
2. [Disk preparation](#disk-preparation)
3. [Partitioning](#partitioning)
4. [LUKS2 encryption](#luks2-encryption)
5. [Btrfs and subvolumes](#btrfs-and-subvolumes)
6. [Mount options](#mount-options)
7. [Compression](#compression)
8. [TRIM and discard](#trim-and-discard)
9. [Mount the system](#mount-the-system)
10. [Swap space](#swap-space)
11. [fstab](#fstab)
12. [Unlock at boot](#unlock-at-boot)
13. [LVM variant](#lvm-variant)

## The layers

The default installation uses three layers:

```
Physical disk
    │
    ├── GPT
    │
    ├── partition 1 ── ESP (FAT32)  or  BIOS boot + /boot
    │
    └── partition 2 ── LUKS2 (Argon2id)
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

`/boot` stays outside the encrypted container. This decision has an important
result. GRUB reads the kernel and the initramfs from a plain partition, thus
GRUB does not open LUKS. Argon2id runs one time, in the initramfs, where
`cryptsetup` has the memory and the CPU time that the function needs.

The system thus asks for the passphrase one time, after the GRUB menu. A layout
that also encrypts `/boot` sets `GRUB_ENABLE_CRYPTODISK`, and it asks two
times: GRUB asks to read `/boot`, and the initramfs asks again for the root
device. This guide keeps the single prompt.

### The boot chain

LUKS gives confidentiality to everything in the container. GRUB, the kernel and
the initramfs stay outside it, because the machine must read them before it has
the passphrase.

This splits the two security properties. An attacker who takes the disk reads
none of your data. An attacker with repeated physical access can modify the
kernel or the initramfs, and the modified initramfs can capture the passphrase
at the next boot.

Encryption of `/boot` answers that only in part, because GRUB itself stays
readable and modifiable. The complete answer verifies a signature on what the
machine starts:

```
Secure Boot  →  signature check  →  GRUB, kernel, initramfs
                                          ↓
                                    LUKS2 + Argon2id
```

Signed boot components let the guide keep `/boot` in plain form and keep the
single passphrase prompt. That work is on the roadmap.

### Why Btrfs goes directly on LUKS2

Btrfs gives you one storage pool with subvolumes, snapshots, quotas and
multiple device support. An LVM volume group with one logical volume that fills
100 % of the space adds a layer that manages nothing. Btrfs does each resize,
each snapshot and each space reservation itself.

LVM becomes useful when the design needs several independent block devices in
the same encrypted container. An example is one volume for virtual machines,
one volume with a different filesystem, and unassigned space in the group. The
[LVM variant](#lvm-variant) section shows that layout.

## Disk preparation

Find the target disk. Check the device name two times. The commands in this
section destroy all data on that disk.

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

Set the variable to the disk that you selected:

```bash
export DRIVE=/dev/nvme0n1
```

Write random data to the disk before you make the container. An attacker then
cannot tell which sectors hold encrypted data, and which sectors stay unused.
This step is optional. It takes as long as one full write of the disk.

```bash
cryptsetup open --type plain --key-file /dev/urandom $DRIVE wipe
dd if=/dev/zero of=/dev/mapper/wipe status=progress bs=16M
cryptsetup close wipe
```

Zeros that go through a plain encrypted mapping become random data. This
method is much faster than a read of `/dev/urandom`.

Then remove each old partition table:

```bash
sgdisk --zap-all $DRIVE
```

## Partitioning

First, find if the system boots with UEFI or with BIOS:

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

Select **one** of the two layouts.

### UEFI

One EFI system partition and the encrypted container:

```bash
export DISK_LABEL="gentoosys"

sgdisk --clear \
       --new=1:0:+1GiB --typecode=1:ef00 --change-name=1:EFI \
       --new=2:0:0     --typecode=2:8309 --change-name=2:${DISK_LABEL} \
       $DRIVE

export EFI_PART_LABEL=EFI
export LUKS_PART_LABEL=${DISK_LABEL}
```

The type code `8309` identifies a LUKS partition. The code `8300` also works.
But `8309` gives the correct description, and the system tools then know that
the partition holds encrypted data.

One gibibyte for the ESP holds several kernels with their initramfs images.
This includes the images that you make during the boot tests.

### BIOS

Three partitions: the area for the second stage of GRUB, a separate `/boot`,
and the encrypted container.

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

## LUKS2 encryption

`cryptsetup` uses Argon2id as the default key derivation function since
version 2.4.0. RFC 9106 makes Argon2id the primary variant. Argon2id gives
resistance against side-channel attacks and against time-memory trade-offs.

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

`--iter-time 5000` gives five seconds to the key derivation. With that budget,
`cryptsetup` calibrates the memory and the iterations that the machine accepts.
The cost of a brute-force attack thus follows your hardware, not a constant.

The passphrase protects all the other data. Select a passphrase that resists a
dictionary attack. Several random words are better than a short string with
character substitutions.

Open the container:

```bash
cryptsetup open /dev/disk/by-partlabel/${LUKS_PART_LABEL} cryptroot
```

The decrypted device is `/dev/mapper/cryptroot`.

> If the session stops and you start again, open the container one more time
> with this command.

## Btrfs and subvolumes

```bash
export BTRFS_LABEL="gentoobtrfs"
mkfs.btrfs --label ${BTRFS_LABEL} /dev/mapper/cryptroot
```

Mount the top level of the filesystem to make the subvolumes:

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

Each subvolume has a purpose:

| Subvolume    | Mount point        | Purpose                                     |
|--------------|--------------------|---------------------------------------------|
| `@`          | `/`                | The root of the installed system.           |
| `@home`      | `/home`            | Keeps user data through a reinstallation.   |
| `@snapshots` | `/.snapshots`      | Holds the snapshots.                        |
| `@varlog`    | `/var/log`         | Keeps the logs out of the snapshots.        |
| `@vartmp`    | `/var/tmp`         | Isolates the temporary files.               |
| `@portage`   | `/var/tmp/portage` | Gets its own mount options.                 |
| `@swap`      | `/var/swap`        | Holds the swapfile.                         |

`@portage` needs an explanation. Portage writes and deletes tens of gibibytes
during a large build. Those files go away some minutes later. A separate
subvolume lets you mount it with cheap compression, or with no compression,
while the other subvolumes keep the aggressive setting.

## Mount options

The default configuration of the guide:

```
noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@
```

And for `@portage`:

```
noatime,compress=zstd:1,space_cache=v2,nodiscard,subvol=@portage
```

Three notes on how the guide selected that list:

**Btrfs finds the device type itself.** It reads the topology of the block
device and finds if the device is rotational. The guide leaves that work to
Btrfs. Set `ssd` by hand only if a virtual layer shows the property
incorrectly. The old layout optimisations for SSDs went away, because they gave
no benefit on current hardware and could increase fragmentation.

**The guide writes the options it wants.** `defaults` groups options that are
already active, thus the list stays short and each entry carries meaning.

**`space_cache=v2` is explicit,** although it is now the default behaviour. The
guide states the intended configuration, and that statement stays stable as the
kernel defaults change. The one cost is compatibility: write operations need a
kernel with free space tree support, which each modern kernel has.

## Compression

`compress-force` tells Btrfs to try compression on each extent. If the
compressed result is larger than the input, Btrfs writes the data without
compression. Thus no file becomes larger. The cost is the CPU time of an
unsuccessful try.

The Btrfs documentation recommends plain `compress` for general use, because
the current heuristics are good. This guide selects `compress-force` for two
reasons.
The first reason is coverage. A file can start with incompressible data and
continue with very compressible data. Without `force`, an early decision can
exclude all the data that comes after it. The second reason is a preference for
an explicit policy over a heuristic that changes between kernel versions.

### Select the level

Btrfs puts the zstd levels in three groups. Levels 1 to 3 are near real time.
Levels 4 to 8 are slower and give a better ratio. Levels 9 to 15 use much more
effort, and the decrease in size can be small.

One asymmetric property decides the choice. **The cost to compress increases
with the level. The cost to decompress stays almost constant.** High levels
thus make writes slower and leave reads almost unchanged.

| Level     | Role in the guide  | When it applies                                                        |
|-----------|--------------------|------------------------------------------------------------------------|
| `zstd:5`  | Default            | A modern workstation: a good balance between space and CPU.            |
| `zstd:9`  | High compression   | Much CPU, and storage that is more valuable than write time.           |
| `zstd:15` | Maximum compression| Stable data that you write one time and read many times.               |

`zstd:5` is the default of this guide. It compresses more than the Btrfs
default and keeps a reasonable write performance on current hardware.

`zstd:9` and `zstd:15` are fully valid options. The workload decides, not the
speed of the machine. A processor with many cores does not make `zstd:15`
correct for `/var/tmp/portage`. There, you spend CPU on files that you delete
some minutes later, in competition with the build. For a stable data set that
stays on disk for months, the result is different.

For this reason, `@portage` uses `compress=zstd:1`. It is the cheapest
compression, without `force`, thus Portage and Clang get the cores.

## TRIM and discard

TRIM tells the device which blocks no longer hold useful data. Without that
information, the SSD controller operates as if it has much less free space than
it really has. This degrades the garbage collection and the sustained
performance.

The cost is privacy. The kernel documentation says that discard through
dm-crypt shows information about the volume: which regions are free, how much
space you use, and some filesystem properties. The key, the file names and the
file contents stay protected. The allocation pattern is what an attacker sees.

TRIM must go through all the layers. Therefore you make the decision in two
places: in Btrfs and in dm-crypt.

| Profile           | dm-crypt         | Btrfs            | Priority                       |
|-------------------|------------------|------------------|--------------------------------|
| **A** Maximum privacy | no discard   | `nodiscard`      | Opacity of the allocation pattern |
| **B** Balanced    | `allow-discards` | `nodiscard` + weekly `fstrim` | Time privacy and SSD health |
| **C** Performance | `allow-discards` | `discard=async`  | Sustained SSD consistency      |

**This guide uses profile B.**

Profile A keeps the maximum opacity. But it gives up all knowledge of free
space in the SSD. On a Gentoo machine the cost increases with time. Each large
build writes and discards tens of gibibytes. The controller then manages a disk
that it believes is much more full than it is.

Profile C gives the device an almost continuous picture of the released space.
Btrfs groups the extents and limits the operations, thus the effect on
performance is small. In exchange, an attacker with access to the storage gets
current information about what became free, and when.

Profile B does the TRIM in batches while the machine is idle. It gets almost
all the practical benefit of C. The `fstrim` manual page says that one weekly
run is enough for a usual desktop or server. Profile B also removes the time
resolution. It does not announce each release at the moment it occurs. It sends
one large set one time each week. The free space pattern becomes visible in the
end. The timeline is what you keep private.

Set `allow-discards` on the kernel command line, with the other dracut
parameters:

```
rd.luks.allow-discards
```

Then schedule the periodic trim with cron:

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

To use profile C, replace `nodiscard` with `discard=async` in the mount options
and do not install the cron job. To use profile A, remove
`rd.luks.allow-discards` from the kernel command line.

## Mount the system

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

Then mount the boot partition.

For UEFI:

```bash
mkfs.vfat -F 32 -n EFI /dev/disk/by-partlabel/${EFI_PART_LABEL}
mount -o rw,nosuid,nodev,noexec,relatime,fmask=0077,dmask=0077 \
      /dev/disk/by-partlabel/${EFI_PART_LABEL} /mnt/gentoo/boot
```

The masks `0077` limit the partition to `root`. The ESP does not support POSIX
permissions, thus you apply the limit at mount time. Without the masks, each
user can read the kernel and the initramfs.

For BIOS:

```bash
mkfs.ext4 -L BOOT /dev/disk/by-partlabel/${BOOT_PART_LABEL}
mount -o rw,nosuid,nodev,relatime \
      /dev/disk/by-partlabel/${BOOT_PART_LABEL} /mnt/gentoo/boot
```

## Swap space

Since version 6.1 of `btrfs-progs`, Btrfs has a command that makes the swapfile
with the correct attributes. The file gets no copy-on-write and no compression.
The command also runs `mkswap` on the file.

```bash
btrfs filesystem mkswapfile --size 8G /mnt/gentoo/var/swap/swapfile
swapon /mnt/gentoo/var/swap/swapfile
```

That one command replaces the sequence of `truncate`, `chattr +C`, `dd` and
`mkswap`.

Use this table for the size:

| RAM    | Without hibernation | With hibernation |
|--------|---------------------|------------------|
| 4 GiB  | 2 GiB               | 6 GiB            |
| 8 GiB  | 3 GiB               | 11 GiB           |
| 16 GiB | 4 GiB               | 20 GiB           |
| 32 GiB | 6 GiB               | 38 GiB           |
| 64 GiB | 8 GiB               | 72 GiB           |

### Hibernation

Hibernation needs the physical offset of the file in the filesystem. That value
is **not** the value that `filefrag` prints. `btrfs-progs` calculates it:

```bash
btrfs inspect-internal map-swapfile -r /mnt/gentoo/var/swap/swapfile
```

Give that number to the kernel as `resume_offset`. Also set `resume=` to the
decrypted device. [installation.md](installation.md) shows the full
configuration.

## fstab

Refer to the swapfile **by path**. The UUID from `mkswap` identifies the swap
"filesystem". The UUID stays inside a file, thus it is not visible and not
usable as a device identifier. A `UUID=` entry for a swapfile does not work.

```bash
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
export BOOT_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${EFI_PART_LABEL})
```

```
# <filesystem>           <mount point>       <type>  <options>                                                      <dump> <pass>

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

The sixth column is `0` for each Btrfs entry. `fsck` does no work on Btrfs.
Btrfs checks its checksums when it reads the data.

## Unlock at boot

The initramfs opens the LUKS container before it mounts the root filesystem.
The kernel parameters from dracut control this. With that configuration, the
OpenRC `dmcrypt` service does no work on the root device.

You need `dmcrypt` only for more encrypted volumes that open after boot. In
OpenRC you configure it in `/etc/conf.d/dmcrypt`. The syntax is different from
the syntax of `/etc/crypttab`:

```bash
# /etc/conf.d/dmcrypt
target=data
source='UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
```

```bash
rc-update add dmcrypt boot
```

## LVM variant

LVM becomes useful when the encrypted container must hold several independent
block devices. You make the volume group on the decrypted device:

```
GPT
 └── LUKS2
       └── LVM2 (volume group)
             ├── lv_root  → Btrfs   →  /
             ├── lv_vm    → XFS     →  virtual machine images
             └── free space in reserve
```

```bash
pvcreate /dev/mapper/cryptroot
vgcreate vg_gentoo /dev/mapper/cryptroot

lvcreate --size 200G --name lv_root vg_gentoo
lvcreate --size 300G --name lv_vm   vg_gentoo

mkfs.btrfs --label gentoobtrfs /dev/vg_gentoo/lv_root
mkfs.xfs -L vmstore /dev/vg_gentoo/lv_vm
```

Keep some space unassigned in the group if you plan to make a volume larger
later. That flexibility is what LVM gives you and Btrfs does not.

With this variant, the root filesystem moves from `/dev/mapper/cryptroot` to
`/dev/vg_gentoo/lv_root`. Change the mounts, the `fstab` and the kernel command
line. The kernel command line also needs this parameter:

```
rd.lvm.lv=vg_gentoo/lv_root
```

The initramfs must include the `lvm` module. The OpenRC `lvm` service must be
in the `boot` runlevel:

```bash
rc-update add lvm boot
```

If your design has one logical volume that fills the group, LVM manages
nothing. Use the default layout of this guide.

---

> 🌐 **Language:** [Latina](../la/receptaculum.md) · [Español](../es/almacenamiento.md) · **English**
