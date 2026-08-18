<!--
i18n/en/installation.md
@fraxgut
CC-BY-SA-4.0
The installation procedure from the live medium to the first boot
-->

# Installation

> 🌐 **Language:** **English** · [Español](../es/installation.md)

This is the full procedure, from the installation medium to the first boot of
the installed system. The storage design is in [storage.md](storage.md). The
toolchain is in [toolchain.md](toolchain.md).

## Contents

1. [Before you start](#before-you-start)
2. [The installation environment](#the-installation-environment)
3. [The disk](#the-disk)
4. [Values you replace](#values-you-replace)
5. [The stage3](#the-stage3)
6. [Enter the chroot](#enter-the-chroot)
7. [Portage and the profile](#portage-and-the-profile)
8. [Locales with musl](#locales-with-musl)
9. [The kernel](#the-kernel)
10. [The initramfs](#the-initramfs)
11. [The bootloader](#the-bootloader)
12. [Network, users and services](#network-users-and-services)
13. [First boot](#first-boot)
14. [After the installation](#after-the-installation)

## Before you start

This guide assumes experience with Gentoo. You must know `emerge`, USE flags,
profiles and manual kernel configuration. You must also know partitioning,
LUKS and the filesystems in this design.

The result uses musl as the C library, OpenRC as the init system, Clang and
LLD as the toolchain, and the Zen kernel. None of these is the Gentoo default.
Several of them are in profiles that Gentoo marks as experimental.

You need a Linux environment to install from. The official Gentoo image works.
This guide uses [SystemRescue](https://www.system-rescue.org/) for its tools.

Write the image to a USB device from another Linux machine:

```bash
lsblk -o NAME,SIZE,MODEL          # find the correct device
export USB=/dev/sdX               # replace X
dd if=systemrescue.iso of=${USB} bs=4M status=progress oflag=sync
```

Warning: this command erases the device. Check the letter before you run it.

## The installation environment

Boot from the medium. Then make the network connection:

```bash
nmtui                             # configure the connection, if necessary
ping -c3 gentoo.org
```

Set the clock. An incorrect time makes the signature check and the certificate
check fail:

```bash
chronyd -q 'pool pool.ntp.org iburst'
```

Work through SSH from another machine. You can then copy and paste the
commands, and you make fewer transcription errors:

```bash
passwd                            # temporary root password
systemctl start sshd              # SystemRescue uses systemd
ip -brief addr                    # shows the address to connect to
```

## The disk

[storage.md](storage.md) gives the partitioning, the LUKS2 container, the Btrfs
subvolumes and the swapfile. Do that procedure before you continue.

At the end of that section, the root filesystem must be at `/mnt/gentoo`.
`/boot` and the other subvolumes must be in position:

```bash
findmnt -R /mnt/gentoo
```

## Values you replace

Two kinds of value appear in the commands of this guide.

**Values that describe your machine.** Each one stays wrong until you change
it:

| Value              | What it is                                    | Where it appears              |
|--------------------|-----------------------------------------------|-------------------------------|
| `/dev/nvme0n1`     | The target disk. A wrong name erases the wrong disk. | `DRIVE`                |
| `/dev/sdX`         | The USB device for the installation medium    | The live medium step          |
| `yourhostname`     | The name of the machine                       | `/etc/hostname`, `/etc/hosts` |
| `yourusername`     | Your login name                               | `useradd`                     |
| `America/Santiago` | Your time zone                                | `/etc/env.d/00musl`           |
| `en_GB.UTF-8`      | Your locale                                   | `/etc/env.d/00musl`           |
| `-j16 -l16`        | Your CPU thread count, from `nproc`           | `MAKEOPTS`                    |

**Names that this guide chooses.** They work as they are. If you change one,
change it everywhere it appears, because later steps read the name back:

| Name          | What it names                        | Read back by                         |
|---------------|--------------------------------------|--------------------------------------|
| `cryptroot`   | The unlocked LUKS mapping            | fstab, kernel command line, recovery |
| `gentoosys`   | The partition label of the container | `LUKS_PART_LABEL`, recovery          |
| `gentoobtrfs` | The Btrfs filesystem label           | `BTRFS_LABEL`                        |
| `EFI`, `BOOT` | The boot partition labels            | Mount steps, fstab, recovery         |
| `vg_gentoo`   | The volume group of the LVM variant  | Kernel command line                  |

The guide exports the first group into shell variables, thus you set each value
one time. Set them again if you leave the shell and return.

## The stage3

This guide uses the `amd64-musl-llvm-openrc` stage3. That file sets three
properties that you cannot change later without a rebuild of the system. The C
library is musl. The toolchain is LLVM. The init system is OpenRC.

Gentoo publishes a manifest for the most recent build. Thus you do not copy a
date into the commands:

```bash
cd /mnt/gentoo

export MIRROR="https://distfiles.gentoo.org/releases/amd64/autobuilds"
export STAGE3_PATH=$(curl -s "${MIRROR}/latest-stage3-amd64-musl-llvm-openrc.txt" \
                     | grep -v '^#' | grep 'tar.xz' | cut -d' ' -f1)
export STAGE3_FILE=$(basename "${STAGE3_PATH}")

echo "${STAGE3_FILE}"
```

Download the file, its signature and its checksums:

```bash
curl -O "${MIRROR}/${STAGE3_PATH}"
curl -O "${MIRROR}/${STAGE3_PATH}.asc"
curl -O "${MIRROR}/${STAGE3_PATH}.DIGESTS"
```

### Check the file

The two checks have different functions. The signature shows that the file
comes from Gentoo. The checksums show that the file arrived complete. You need
both.

```bash
# Import the Release Engineering keys
curl -s https://qa-reports.gentoo.org/output/service-keys.gpg | gpg --import

# Check the signature of the tarball
gpg --verify "${STAGE3_FILE}.asc" "${STAGE3_FILE}"

# Check the SHA-256 sum
grep -A1 'SHA256' "${STAGE3_FILE}.DIGESTS" | grep "${STAGE3_FILE}$"
sha256sum "${STAGE3_FILE}"
```

`gpg` must print `Good signature`. The two sums must agree character by
character. If one check fails, delete the file and download it again.

### Unpack the file

```bash
tar xpvf "${STAGE3_FILE}" --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```

`--xattrs-include` keeps the extended attributes. The capabilities of the
binaries need them. `--numeric-owner` stops the users of the live environment
from a change of the file ownership.

## Enter the chroot

Copy the DNS configuration. Then mount the virtual filesystems:

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys  /mnt/gentoo/sys  && mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev  /mnt/gentoo/dev  && mount --make-rslave /mnt/gentoo/dev
mount --bind  /run  /mnt/gentoo/run  && mount --make-slave  /mnt/gentoo/run
```

Enter:

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

> Each time that you leave the chroot and enter it again, do these three
> commands. Also export the variables that you set before.

## Portage and the profile

Get the package tree:

```bash
emerge-webrsync
```

Select the profile. This guide uses its own profile. That profile puts the
`hardened` features on `musl/llvm`. [toolchain.md](toolchain.md) shows how to
make it. For now, select the base profile:

```bash
eselect profile list | grep musl
eselect profile set default/linux/amd64/23.0/musl/llvm
```

Write `/etc/portage/make.conf` with the configuration from
[toolchain.md](toolchain.md). Then add the CPU flags and the mirrors:

```bash
emerge app-portage/cpuid2cpuflags app-portage/mirrorselect

echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
mirrorselect -i -o >> /etc/portage/make.conf
```

`package.use/00cpu-flags` is better than a `CPU_FLAGS_X86` line in `make.conf`.
Portage manages the variable per package. You also make the file again with one
command when the hardware changes.

### Package branches

The base system stays on the stable branch. Packages that exist only in the
testing branch get an entry each. Do not open `~amd64` for the full system:

```bash
mkdir -p /etc/portage/package.accept_keywords
echo "sys-kernel/zen-sources ~amd64" >> /etc/portage/package.accept_keywords/kernel
```

Apply the profile and the configuration to the installed system:

```bash
emerge --ask --verbose --update --deep --newuse @world
```

## Locales with musl

musl does not have the locale mechanism of glibc. Gentoo has
`sys-apps/musl-locales` in the main repository, and it is stable on amd64:

```bash
emerge sys-apps/musl-locales sys-libs/timezone-data
```

Set the time zone and the locale in `/etc/env.d`:

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

`LC_COLLATE="C"` sorts by code point, not by language rules. It is faster, and
it makes the sort order in scripts predictable. Change it if you need the sort
order of your language.

## The kernel

Install the firmware, the sources and the boot tools:

```bash
emerge sys-kernel/linux-firmware sys-kernel/zen-sources
emerge sys-kernel/dracut
```

`sys-kernel/installkernel` does the work after the build. It makes the
initramfs and it updates the bootloader each time that you install a kernel.

```bash
echo "sys-kernel/installkernel dracut grub" \
    >> /etc/portage/package.use/installkernel
emerge sys-kernel/installkernel
```

Select the sources:

```bash
eselect kernel list
eselect kernel set 1
cd /usr/src/linux
```

### Configuration

The configuration depends on your hardware. Examine it with `lspci -k`, `lsusb`
and `lscpu`. Then enable what you need. These options are necessary for a boot
with this storage design:

| Area                  | Option                                                    |
|-----------------------|-----------------------------------------------------------|
| Device Drivers        | `Device mapper support` and `Crypt target support`        |
| File systems          | `Btrfs filesystem support`                                |
| File systems          | `VFAT` for the ESP, or `ext4` for `/boot` on BIOS         |
| Cryptographic API     | `AES`, `XTS`, `SHA512`                                    |
| General setup         | Initramfs support and the compression that dracut uses    |
| Storage               | The driver for your disks: NVMe, AHCI or SCSI             |

Without `Crypt target support`, the initramfs cannot open the container.
Without the disk driver, it cannot find the disk. These two are the most
frequent failures at the first boot.

```bash
make menuconfig
```

### Build

Clang, LLD and the integrated assembler build the kernel:

```bash
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

The last command calls `installkernel`. That tool makes the initramfs and
writes the GRUB configuration again.

## The initramfs

Configure dracut before you install the kernel. `installkernel` then makes a
complete initramfs:

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

If you use the LVM variant, add `lvm` to `add_dracutmodules`.

Check the result:

```bash
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

## The bootloader

GRUB 2.14 carries Argon2 support upstream, and it is stable on amd64. Emerge
it, and it works with the LUKS2 container as it is.

`/boot` stays outside the encrypted container. GRUB reads the kernel and the
initramfs in plain form and starts them. The initramfs then opens LUKS. The
system thus asks for the passphrase one time, after the GRUB menu:

```
firmware → GRUB → kernel + initramfs → passphrase → Btrfs root → OpenRC
```

A layout that also encrypts `/boot` asks for the passphrase two times. GRUB
asks first, to read `/boot`, and the initramfs asks again. The two programs run
in separate environments, and GRUB gives the kernel no unlocked session. Both
prompts take the same passphrase and open the same keyslot. This guide keeps
the single prompt, and `GRUB_ENABLE_CRYPTODISK` stays out of the configuration.

```bash
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf   # or "pc" on BIOS
emerge sys-boot/grub
```

Collect the identifiers for the kernel command line:

```bash
export LUKS_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${LUKS_PART_LABEL})
export RESUME_OFFSET=$(btrfs inspect-internal map-swapfile -r /var/swap/swapfile)
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
```

Configure `/etc/default/grub`:

```ini
# rd.luks.uuid           identifies the container that the initramfs opens
# rd.luks.allow-discards lets TRIM go through dm-crypt (profile B)
# root, rootflags        the root device and the subvolume to mount on it
# resume, resume_offset  hibernation onto the swapfile
GRUB_CMDLINE_LINUX="rd.luks.uuid=${LUKS_UUID} rd.luks.allow-discards root=UUID=${ROOT_UUID} rootflags=subvol=@ resume=UUID=${ROOT_UUID} resume_offset=${RESUME_OFFSET}"
GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3"
```

Install GRUB.

For UEFI:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
```

For BIOS:

```bash
grub-install --target=i386-pc $DRIVE
```

Then make the configuration:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Read the output. GRUB must find the kernel and its initramfs. The menu entry
must contain the parameters above.

## Network, users and services

### The hostname

The name identifies the machine on each network that it joins, thus it must be
unique on those networks. Two machines with the same name on one LAN collide,
and mDNS renames the second to `name-2`. The name needs no global uniqueness.

RFC 1123 gives the syntax:

- 1 to 63 characters.
- Letters, digits and the hyphen. A digit can start the name.
- A hyphen never starts the name and never ends it.
- No dots and no underscores. A dot separates the labels of a domain name.

Lowercase is the convention, because DNS ignores case.

The OpenRC `hostname` service reads `/etc/hostname` first. It uses the
`hostname` variable in `/etc/conf.d/hostname` when that file is empty.

```bash
printf '%s\n' "yourhostname" > /etc/hostname
```

OpenRC needs the same name in `/etc/hosts`. That entry lets the machine resolve
its own name through a local lookup:

```
127.0.0.1   yourhostname.localdomain yourhostname localhost
::1         yourhostname.localdomain yourhostname localhost
```

### Users

```bash
passwd                                   # permanent root password

useradd -m -G wheel,users,audio,video,portage yourusername
passwd yourusername
```

The `wheel` group gets root privileges through `doas`. The `portage` group
lets the account read the build directories.

Install the network manager and the system services:

```bash
emerge net-misc/networkmanager app-admin/metalog net-misc/chrony
emerge app-admin/doas sys-process/cronie

rc-update add NetworkManager default
rc-update add metalog default
rc-update add chronyd default
rc-update add cronie default
```

`cronie` runs the weekly trim from [storage.md](storage.md).

Let the `wheel` group get root privileges:

```bash
echo "permit persist :wheel" > /etc/doas.conf
chmod 0400 /etc/doas.conf
```

You need the `dmcrypt` service only for more encrypted volumes that open after
boot. The initramfs opens the root device.

## First boot

Leave the chroot. Then unmount in the opposite sequence:

```bash
exit
cd /
swapoff /mnt/gentoo/var/swap/swapfile
umount -R /mnt/gentoo
cryptsetup close cryptroot
reboot
```

Remove the installation medium. GRUB must appear. The initramfs must ask for
the LUKS passphrase before it mounts the root filesystem.

If something fails, [troubleshooting.md](troubleshooting.md) gives the usual
causes.

## After the installation

The system now boots with musl, OpenRC, Clang and the Zen kernel. Two tasks
stay open: the stacked profile with the hardened features, and the rebuild of
the system with LTO. [toolchain.md](toolchain.md) shows both.

Do that work on the installed system, not in the chroot. If the rebuild breaks
something, you then have a system that boots, and you can repair it from there.

---

> 🌐 **Language:** **English** · [Español](../es/installation.md)
