<!--
i18n/en/troubleshooting.md
@guterion
CC-BY-SA-4.0
Recovery procedures for boot, storage and build failures
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/remedia.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/problemas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **English**

# Troubleshooting

</div>

---

The failures of this installation occur at three moments: the boot, the mount
of the storage, and the build. This page groups them by symptom.

## Contents

1. [Return to the installed system](#return-to-the-installed-system)
2. [GRUB does not appear](#grub-does-not-appear)
3. [GRUB appears but the boot stops](#grub-appears-but-the-boot-stops)
4. [The root mounts but subvolumes are absent](#the-root-mounts-but-subvolumes-are-absent)
5. [Hibernation does not restore](#hibernation-does-not-restore)
6. [Build failures](#build-failures)
7. [The stacked profile has no effect](#the-stacked-profile-has-no-effect)
8. [No network after a reboot](#no-network-after-a-reboot)

## Return to the installed system

Almost all repairs need a chroot into the installed system. Boot from the
installation medium. Then make the mounts again:

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

With the LVM variant, activate the volume group first:

```bash
vgchange -ay
```

## GRUB does not appear

The firmware does not find the bootloader. Check these items in sequence:

**The installation target.** UEFI and BIOS use different targets. An incorrect
target installs a bootloader that the firmware cannot run:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-install --target=i386-pc /dev/nvme0n1
```

**The firmware entry.** On UEFI, check that the entry exists and that it is in
the boot sequence:

```bash
efibootmgr -v
```

If the entry is absent, and the firmware continues to ignore it after you make
it again, install to the default boot path. Each UEFI firmware accepts that
path:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --removable
```

**The format of the ESP.** It must be FAT32. With a different filesystem, the
firmware cannot read the partition.

## GRUB appears but the boot stops

The symptom shows the cause.

### It does not ask for the passphrase

The initramfs does not have what it needs to open the container. Make it again
and check its contents:

```bash
dracut --force --kver "$(uname -r)"
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

If `crypt` is absent, examine `/etc/dracut.conf.d/10-gentoo.conf`. If `crypt`
is present and the kernel panics, the kernel configuration has no
`Crypt target support` or no `Device mapper support`. Dracut included the
module, but the kernel gives no target for it.

### It asks for the passphrase but does not find the root

Examine the kernel command line that GRUB made:

```bash
grep -o 'rd.luks[^ ]*\|root=[^ ]*\|rootflags=[^ ]*' /boot/grub/grub.cfg | sort -u
```

`rd.luks.uuid` must be the UUID of the LUKS **partition**. `root` must be the
UUID of the **decrypted device**. They are two different identifiers, and to
confuse them is a frequent error:

```bash
blkid -s UUID -o value /dev/disk/by-partlabel/gentoosys   # rd.luks.uuid
blkid -s UUID -o value /dev/mapper/cryptroot              # root
```

After you correct `/etc/default/grub`, make the configuration again:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### The keyboard does not work at the passphrase prompt

The initramfs has no keyboard drivers. Add them and make the image again:

```
add_drivers+=" hid_generic usbhid xhci_hcd "
```

### It does not find the disk

The storage driver is absent. With `hostonly="yes"`, dracut includes only the
modules for the hardware that is present at build time. If the disk changed, or
if you made the image on another machine, build it again:

```bash
dracut --force --no-hostonly --kver "$(uname -r)"
```

## The root mounts but subvolumes are absent

Examine `/etc/fstab`. Each Btrfs entry uses the UUID of the decrypted device,
and `subvol=` makes them different:

```bash
findmnt --verify --verbose
btrfs subvolume list /
```

The mount points must exist in the `@` subvolume. An absent directory makes the
mount fail without a message at boot:

```bash
mkdir -p /home /.snapshots /var/log /var/tmp /var/swap /var/tmp/portage
```

## Hibernation does not restore

The offset of the swapfile changes if you make the file again. Calculate it one
more time and correct the kernel command line:

```bash
btrfs inspect-internal map-swapfile -r /var/swap/swapfile
```

That value is **not** the value that `filefrag` prints. Also, `resume` must
point to the decrypted device, not to the LUKS partition, and the initramfs
needs the `resume` module.

## Build failures

### The failure occurs at link time

LTO finds incompatibilities at link time that stayed hidden before. Disable LTO
for the package that fails, not for the full system:

```bash
echo "category/package no-lto" >> /etc/portage/package.env/no-lto
```

[toolchain.md](toolchain.md) has the definition of the `no-lto` environment.

### The package does not build with Clang

Not all software accepts Clang. First, look for the failure in the Gentoo bug
tracker. As a local measure, build the package with GCC if you have it
installed, or lower the optimisation level:

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

### The build runs out of memory

LTO and Clang use much more memory than a usual build. Each parallel job
multiplies that memory. Decrease the parallel jobs before you decrease the
optimisation level:

```ini
MAKEOPTS="-j4 -l8"
```

`/var/tmp/portage` on tmpfs makes the problem worse, because the temporary
directory and the compiler compete for the same memory. The design of this
guide keeps that directory on disk, in its own subvolume.

### The LLVM JIT is absent

The `hardened` profile sets `-jit -orc`. If a package needs them — Mesa is the
usual example — enable them for that package alone:

```bash
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

## The stacked profile has no effect

Examine what Portage reads:

```bash
eselect profile show
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

If the flags are absent, the cause is usually `layout.conf`. The
`repository:path` syntax in the `parent` file needs the declared profile
format:

```bash
grep profile-formats /var/db/repos/local/metadata/layout.conf
```

```ini
profile-formats = portage-2
```

`eselect repository create` does not always write that line. Add it. Then
select the profile again.

## No network after a reboot

```bash
rc-status                        # does NetworkManager run?
rc-update show | grep -i network # is it in the default runlevel?
ip -brief link                   # does the kernel know the interface?
```

If the interface is absent, the kernel has no driver for it. Find the device
and the module that it needs:

```bash
lspci -k | grep -A3 -i net
```

Then configure the connection with `nmtui`.

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/remedia.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **[Español](../es/problemas.md)** · <img src="../../assets/flag-england.svg" alt="" height="14"> **English**

</div>
