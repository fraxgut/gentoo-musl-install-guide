<!--
i18n/es/installation.md
@fraxgut
CC-BY-SA-4.0
The installation procedure from the live medium to the first boot
-->

# Instalación

> 🌐 **Idioma:** [English](../en/installation.md) · **Español**

Procedimiento completo, desde el medio de instalación hasta el primer arranque
del sistema instalado. El diseño de almacenamiento está en
[storage.md](storage.md) y la cadena de herramientas en
[toolchain.md](toolchain.md).

## Índice

1. [Antes de empezar](#antes-de-empezar)
2. [El entorno de instalación](#el-entorno-de-instalación)
3. [El disco](#el-disco)
4. [El stage3](#el-stage3)
5. [Entrar al chroot](#entrar-al-chroot)
6. [Portage y el perfil](#portage-y-el-perfil)
7. [Localización con musl](#localización-con-musl)
8. [El núcleo](#el-núcleo)
9. [El initramfs](#el-initramfs)
10. [El cargador de arranque](#el-cargador-de-arranque)
11. [Red, usuarios y servicios](#red-usuarios-y-servicios)
12. [Primer arranque](#primer-arranque)
13. [Después de instalar](#después-de-instalar)

## Antes de empezar

La guía asume experiencia previa con Gentoo: compilación con `emerge`, banderas
USE, perfiles y configuración manual del núcleo. Asume también soltura con el
particionado, con LUKS y con los sistemas de archivos implicados.

El sistema resultante usa musl como biblioteca C, OpenRC como sistema de
inicio, Clang y LLD como cadena de herramientas, y el núcleo Zen. Ninguna de
esas piezas es la opción por defecto de Gentoo, y varias viven en perfiles
marcados como experimentales.

Necesitas un entorno Linux funcional para instalar. Sirve la imagen oficial de
Gentoo; esta guía usa [SystemRescue](https://www.system-rescue.org/) por sus
herramientas.

Graba la imagen en un dispositivo USB desde otra máquina Linux:

```bash
lsblk -o NAME,SIZE,MODEL          # identifica el dispositivo correcto
export USB=/dev/sdX               # sustituye X
dd if=systemrescue.iso of=${USB} bs=4M status=progress oflag=sync
```

La orden borra el dispositivo indicado. Comprueba la letra antes de ejecutarla.

## El entorno de instalación

Arranca desde el medio y establece la conexión de red:

```bash
nmtui                             # configura la conexión, si hace falta
ping -c3 gentoo.org
```

Sincroniza el reloj. Una hora incorrecta invalida la verificación de firmas y
de certificados:

```bash
chronyd -q 'pool pool.ntp.org iburst'
```

Trabajar por SSH desde otra máquina permite copiar y pegar las órdenes, lo que
reduce los errores de transcripción:

```bash
passwd                            # contraseña temporal de root
systemctl start sshd              # SystemRescue usa systemd
ip -brief addr                    # muestra la dirección a la que conectar
```

## El disco

El particionado, el contenedor LUKS2, los subvolúmenes Btrfs y el archivo de
intercambio están descritos paso a paso en [storage.md](storage.md). Complétalo
antes de continuar.

Al terminar esa sección, el sistema de archivos raíz debe estar montado en
`/mnt/gentoo`, con `/boot` y el resto de los subvolúmenes en su sitio:

```bash
findmnt -R /mnt/gentoo
```

## El stage3

La guía usa el stage3 `amd64-musl-llvm-openrc`. Ese archivo fija tres
decisiones que después no se cambian sin reconstruir el sistema: la biblioteca
C es musl, la cadena de herramientas es LLVM y el sistema de inicio es OpenRC.

Gentoo publica un manifiesto con la compilación más reciente, de modo que no
hace falta copiar ninguna fecha:

```bash
cd /mnt/gentoo

export MIRROR="https://distfiles.gentoo.org/releases/amd64/autobuilds"
export STAGE3_PATH=$(curl -s "${MIRROR}/latest-stage3-amd64-musl-llvm-openrc.txt" \
                     | grep -v '^#' | grep 'tar.xz' | cut -d' ' -f1)
export STAGE3_FILE=$(basename "${STAGE3_PATH}")

echo "${STAGE3_FILE}"
```

Descarga el archivo junto con su firma y sus sumas de verificación:

```bash
curl -O "${MIRROR}/${STAGE3_PATH}"
curl -O "${MIRROR}/${STAGE3_PATH}.asc"
curl -O "${MIRROR}/${STAGE3_PATH}.DIGESTS"
```

### Verificación

La verificación cumple dos funciones distintas: la firma acredita que el
archivo procede de Gentoo, y las sumas acreditan que llegó íntegro. Ambas
importan.

```bash
# Importa las claves de Release Engineering
curl -s https://qa-reports.gentoo.org/output/service-keys.gpg | gpg --import

# Comprueba la firma del tarball
gpg --verify "${STAGE3_FILE}.asc" "${STAGE3_FILE}"

# Comprueba la suma SHA-256
grep -A1 'SHA256' "${STAGE3_FILE}.DIGESTS" | grep "${STAGE3_FILE}$"
sha256sum "${STAGE3_FILE}"
```

`gpg` debe informar `Good signature`, y las dos sumas deben coincidir carácter
a carácter. Si alguna comprobación falla, descarta el archivo y descárgalo de
nuevo.

### Extracción

```bash
tar xpvf "${STAGE3_FILE}" --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```

`--xattrs-include` conserva los atributos extendidos, de los que dependen las
capacidades de los binarios; `--numeric-owner` evita que los usuarios del
entorno live reasignen la propiedad de los archivos.

## Entrar al chroot

Copia la configuración de DNS y monta los sistemas de archivos virtuales:

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys  /mnt/gentoo/sys  && mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev  /mnt/gentoo/dev  && mount --make-rslave /mnt/gentoo/dev
mount --bind  /run  /mnt/gentoo/run  && mount --make-slave  /mnt/gentoo/run
```

Entra:

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

> Cada vez que salgas y vuelvas a entrar al chroot debes repetir estos tres
> comandos y volver a exportar las variables que definiste antes.

## Portage y el perfil

Obtén el árbol de paquetes:

```bash
emerge-webrsync
```

Selecciona el perfil. La guía usa un perfil propio que apila las
características de `hardened` sobre `musl/llvm`; su construcción está en
[toolchain.md](toolchain.md). Mientras tanto, selecciona el perfil base:

```bash
eselect profile list | grep musl
eselect profile set default/linux/amd64/23.0/musl/llvm
```

Escribe `/etc/portage/make.conf` con la configuración descrita en
[toolchain.md](toolchain.md), y añade las banderas de CPU detectadas y los
espejos:

```bash
emerge app-portage/cpuid2cpuflags app-portage/mirrorselect

echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
mirrorselect -i -o >> /etc/portage/make.conf
```

`package.use/00cpu-flags` es preferible a escribir `CPU_FLAGS_X86` en
`make.conf`: Portage administra la variable por paquete y el archivo se
regenera con una sola orden cuando cambias de hardware.

### Ramas de paquetes

El sistema base sigue la rama estable. Los paquetes que sólo existen en la rama
de pruebas se habilitan uno por uno, en lugar de abrir `~amd64` para todo el
sistema:

```bash
mkdir -p /etc/portage/package.accept_keywords
echo "sys-kernel/zen-sources ~amd64" >> /etc/portage/package.accept_keywords/kernel
```

Aplica el perfil y la configuración al sistema existente:

```bash
emerge --ask --verbose --update --deep --newuse @world
```

## Localización con musl

musl no incluye el mecanismo de locales de glibc. Gentoo distribuye
`sys-apps/musl-locales`, que está en el árbol principal y es estable en amd64:

```bash
emerge sys-apps/musl-locales sys-libs/timezone-data
```

Configura la zona horaria y la localización en `/etc/env.d`:

```bash
cat > /etc/env.d/00musl <<'EOF'
MUSL_LOCPATH="/usr/share/i18n/locales/musl"
CHARSET="UTF-8"
LANG="es_CL.UTF-8"
LC_COLLATE="C"
TZ="America/Santiago"
EOF

env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
date
```

`LC_COLLATE="C"` ordena por punto de código en lugar de por reglas
lingüísticas. Es más rápido y hace predecible el orden en los guiones; cámbialo
si necesitas el criterio de ordenación de tu idioma.

## El núcleo

Instala el firmware, las fuentes y las herramientas de arranque:

```bash
emerge sys-kernel/linux-firmware sys-kernel/zen-sources
emerge sys-kernel/dracut
```

`sys-kernel/installkernel` automatiza el trabajo posterior a la compilación:
genera el initramfs y actualiza el cargador de arranque cada vez que instalas
un núcleo.

```bash
echo "sys-kernel/installkernel dracut grub" \
    >> /etc/portage/package.use/installkernel
emerge sys-kernel/installkernel
```

Selecciona las fuentes:

```bash
eselect kernel list
eselect kernel set 1
cd /usr/src/linux
```

### Configuración

La configuración depende del hardware. Investígalo con `lspci -k`, `lsusb` y
`lscpu`, y habilita lo que corresponda. Estas opciones son obligatorias para
que el sistema arranque con este diseño de almacenamiento:

| Área                  | Opción                                                    |
|-----------------------|-----------------------------------------------------------|
| Device Drivers        | `Device mapper support` y `Crypt target support`          |
| File systems          | `Btrfs filesystem support`                                |
| File systems          | `VFAT` para la ESP, o `ext4` para `/boot` en BIOS         |
| Cryptographic API     | `AES`, `XTS`, `SHA512`                                    |
| General setup         | Soporte de initramfs y la compresión que use dracut       |
| Almacenamiento        | El controlador de tus discos: NVMe, AHCI o SCSI           |

Sin `Crypt target support` el initramfs no puede abrir el contenedor. Sin el
controlador de disco, no encuentra el disco. Son los dos fallos más frecuentes
en el primer arranque.

```bash
make menuconfig
```

### Compilación

El núcleo se construye con Clang, LLD y el ensamblador integrado:

```bash
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

La última orden invoca `installkernel`, que genera el initramfs y regenera la
configuración de GRUB.

## El initramfs

Configura dracut antes de instalar el núcleo, para que `installkernel` produzca
un initramfs completo:

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

Si usas la variante con LVM, añade `lvm` a `add_dracutmodules`.

Comprueba el resultado:

```bash
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

## El cargador de arranque

GRUB 2.14 incorpora el soporte de Argon2 en el propio proyecto y es estable en
amd64. Basta con emergerlo: funciona con el contenedor LUKS2 tal cual está.

`/boot` queda fuera del contenedor cifrado. GRUB lee el núcleo y el initramfs
en claro y los arranca; el initramfs abre después el contenedor. El sistema
pide así la frase de paso una sola vez, tras el menú de GRUB:

```
firmware → GRUB → núcleo + initramfs → frase de paso → raíz Btrfs → OpenRC
```

Un diseño que cifre también `/boot` pide la frase dos veces: primero GRUB, para
poder leer `/boot`, y después el initramfs. Ambos programas corren en entornos
separados, y GRUB no entrega al núcleo ninguna sesión ya desbloqueada. Las dos
peticiones aceptan la misma frase y abren el mismo keyslot. Esta guía conserva
la petición única, y por eso `GRUB_ENABLE_CRYPTODISK` queda fuera de la
configuración.

```bash
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf   # o "pc" en BIOS
emerge sys-boot/grub
```

Reúne los identificadores que necesita la línea de órdenes del núcleo:

```bash
export LUKS_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${LUKS_PART_LABEL})
export RESUME_OFFSET=$(btrfs inspect-internal map-swapfile -r /var/swap/swapfile)
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
```

Configura `/etc/default/grub`:

```ini
# rd.luks.uuid          identifica el contenedor que abre el initramfs
# rd.luks.allow-discards habilita TRIM a través de dm-crypt (perfil B)
# root, rootflags       la raíz y el subvolumen que se monta en ella
# resume, resume_offset la hibernación sobre el archivo de intercambio
GRUB_CMDLINE_LINUX="rd.luks.uuid=${LUKS_UUID} rd.luks.allow-discards root=UUID=${ROOT_UUID} rootflags=subvol=@ resume=UUID=${ROOT_UUID} resume_offset=${RESUME_OFFSET}"
GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3"
```

Instala GRUB.

Para UEFI:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
```

Para BIOS:

```bash
grub-install --target=i386-pc $DRIVE
```

Y genera la configuración:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Revisa la salida: debe encontrar el núcleo y su initramfs, y la entrada
generada debe contener los parámetros anteriores.

## Red, usuarios y servicios

El servicio `hostname` de OpenRC lee primero `/etc/hostname`, y recurre a la
variable `hostname` de `/etc/conf.d/hostname` cuando ese archivo está vacío.
Escribe el nombre que quieras:

```bash
echo "tunombredehost" > /etc/hostname
passwd                                   # contraseña definitiva de root

useradd -m -G wheel,users,audio,video,portage tuusuario
passwd tuusuario
```

Instala el gestor de red y los servicios del sistema:

```bash
emerge net-misc/networkmanager app-admin/metalog net-misc/chrony
emerge app-admin/doas sys-process/cronie

rc-update add NetworkManager default
rc-update add metalog default
rc-update add chronyd default
rc-update add cronie default
```

`cronie` ejecuta el recorte semanal descrito en [storage.md](storage.md).

Autoriza al grupo `wheel` a elevar privilegios:

```bash
echo "permit persist :wheel" > /etc/doas.conf
chmod 0400 /etc/doas.conf
```

El servicio `dmcrypt` sólo hace falta si abres volúmenes cifrados adicionales
después del arranque; la raíz la abre el initramfs.

## Primer arranque

Sal del chroot y desmonta en orden inverso:

```bash
exit
cd /
swapoff /mnt/gentoo/var/swap/swapfile
umount -R /mnt/gentoo
cryptsetup close cryptroot
reboot
```

Retira el medio de instalación. GRUB debe aparecer, y el initramfs debe pedir
la frase de paso de LUKS antes de montar la raíz.

Si algo falla, [troubleshooting.md](troubleshooting.md) recorre las causas
habituales.

## Después de instalar

El sistema ya arranca con musl, OpenRC, Clang y el núcleo Zen. Lo que queda es
el perfil apilado con las características de endurecimiento y la reconstrucción
del sistema con LTO, ambos descritos en [toolchain.md](toolchain.md).

Conviene hacerlo con el sistema instalado y arrancando, no dentro del chroot:
si la reconstrucción deja algo inservible, tendrás un sistema que arranca desde
el cual repararlo.

---

> 🌐 **Idioma:** [English](../en/installation.md) · **Español**
