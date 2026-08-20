<!--
i18n/es/troubleshooting.md
@guterion
CC-BY-SA-4.0
Recovery procedures for boot, storage and build failures
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/remedia.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **Español** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/troubleshooting.md)**

# Solución de problemas

</div>

---

Los fallos de esta instalación se concentran en tres momentos: el arranque, el
montaje del almacenamiento y la compilación. Esta página los recorre por
síntoma.

## Índice

1. [Volver al sistema instalado](#volver-al-sistema-instalado)
2. [GRUB no aparece](#grub-no-aparece)
3. [GRUB aparece pero el arranque se detiene](#grub-aparece-pero-el-arranque-se-detiene)
4. [La raíz monta pero faltan subvolúmenes](#la-raíz-monta-pero-faltan-subvolúmenes)
5. [La hibernación no restaura](#la-hibernación-no-restaura)
6. [Fallos de compilación](#fallos-de-compilación)
7. [El perfil apilado no surte efecto](#el-perfil-apilado-no-surte-efecto)
8. [Sin red tras reiniciar](#sin-red-tras-reiniciar)

## Volver al sistema instalado

Casi todas las reparaciones necesitan un chroot sobre el sistema instalado.
Arranca desde el medio de instalación y rehaz el montaje:

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

Con la variante LVM, activa antes el grupo de volúmenes:

```bash
vgchange -ay
```

## GRUB no aparece

El firmware no encuentra el cargador. Comprueba, por orden:

**El destino de la instalación.** UEFI y BIOS usan objetivos distintos, y
equivocarlos instala un cargador que el firmware no puede ejecutar:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-install --target=i386-pc /dev/nvme0n1
```

**La entrada del firmware.** En UEFI, comprueba que existe y que está en el
orden de arranque:

```bash
efibootmgr -v
```

Si la entrada falta y el firmware la sigue ignorando tras recrearla, instala en
la ruta de arranque predeterminada, que todo firmware UEFI acepta:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --removable
```

**El formato de la ESP.** Debe ser FAT32. Otro sistema de archivos deja la
partición ilegible para el firmware.

## GRUB aparece pero el arranque se detiene

El síntoma distingue la causa.

### No pide la frase de paso

El initramfs no incluye lo necesario para abrir el contenedor. Regenéralo y
verifica su contenido:

```bash
dracut --force --kver "$(uname -r)"
lsinitrd /boot/initramfs-*.img | grep -E 'crypt|btrfs'
```

Si `crypt` no aparece, revisa `/etc/dracut.conf.d/10-gentoo.conf`. Si aparece
pero el núcleo entra en pánico igualmente, falta `Crypt target support` o
`Device mapper support` en la configuración del núcleo: dracut incluyó el
módulo, pero el núcleo no ofrece el destino sobre el que operar.

### Pide la frase de paso pero no encuentra la raíz

Revisa la línea de órdenes del núcleo que generó GRUB:

```bash
grep -o 'rd.luks[^ ]*\|root=[^ ]*\|rootflags=[^ ]*' /boot/grub/grub.cfg | sort -u
```

`rd.luks.uuid` debe corresponder al UUID de la **partición** LUKS, y `root` al
UUID del **dispositivo descifrado**. Son dos identificadores distintos y
confundirlos es un error frecuente:

```bash
blkid -s UUID -o value /dev/disk/by-partlabel/gentoosys   # rd.luks.uuid
blkid -s UUID -o value /dev/mapper/cryptroot              # root
```

Tras corregir `/etc/default/grub`, regenera la configuración:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### El teclado no responde al pedir la frase

Faltan los controladores del teclado en el initramfs. Añádelos y regenéralo:

```
add_drivers+=" hid_generic usbhid xhci_hcd "
```

### No encuentra el disco

Falta el controlador de almacenamiento. Con `hostonly="yes"`, dracut incluye
sólo los módulos del hardware presente al generar la imagen; si el disco
cambió, o si la imagen se generó en otra máquina, regenera con:

```bash
dracut --force --no-hostonly --kver "$(uname -r)"
```

## La raíz monta pero faltan subvolúmenes

Revisa `/etc/fstab`. Todas las entradas Btrfs comparten el UUID del dispositivo
descifrado y se distinguen por `subvol=`:

```bash
findmnt --verify --verbose
btrfs subvolume list /
```

Los puntos de montaje deben existir dentro del subvolumen `@`. Un directorio
ausente hace fallar el montaje silenciosamente durante el arranque:

```bash
mkdir -p /home /.snapshots /var/log /var/tmp /var/swap /var/tmp/portage
```

## La hibernación no restaura

El desplazamiento del archivo de intercambio cambia si el archivo se recrea.
Vuelve a calcularlo y actualiza la línea del núcleo:

```bash
btrfs inspect-internal map-swapfile -r /var/swap/swapfile
```

Ese valor **no** es el que informa `filefrag`. Además, `resume` debe apuntar al
dispositivo descifrado, no a la partición LUKS, y el initramfs necesita el
módulo `resume`.

## Fallos de compilación

### El fallo aparece al enlazar

LTO detecta al enlazar incompatibilidades que antes pasaban desapercibidas.
Desactívalo para el paquete afectado en vez de para todo el sistema:

```bash
echo "categoría/paquete no-lto" >> /etc/portage/package.env/no-lto
```

La definición del entorno `no-lto` está en [toolchain.md](herramientas.md).

### El paquete no compila con Clang

No todo el software acepta Clang. Revisa primero si Gentoo ya conoce el fallo
en su gestor de incidencias. Como medida local, el paquete puede construirse
con GCC si lo tienes instalado, o bajar el nivel de optimización:

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

### La compilación agota la memoria

LTO y Clang consumen bastante más memoria que una compilación corriente, y el
consumo se multiplica por cada trabajo en paralelo. Reduce el paralelismo antes
que el nivel de optimización:

```ini
MAKEOPTS="-j4 -l8"
```

`/var/tmp/portage` en tmpfs agrava el problema, porque el directorio temporal
compite con el compilador por la misma memoria. El diseño de esta guía lo
mantiene en disco, en su propio subvolumen.

### Falta el JIT de LLVM

El perfil `hardened` establece `-jit -orc`. Si un paquete los necesita —Mesa es
el caso habitual—, reactívalos sólo para él:

```bash
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

## El perfil apilado no surte efecto

Comprueba qué está viendo Portage:

```bash
eselect profile show
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

Si las banderas no aparecen, la causa suele estar en `layout.conf`. La sintaxis
`repositorio:ruta` del archivo `parent` requiere el formato de perfil
declarado:

```bash
grep profile-formats /var/db/repos/local/metadata/layout.conf
```

```ini
profile-formats = portage-2
```

`eselect repository create` no siempre escribe esa línea. Añádela y vuelve a
seleccionar el perfil.

## Sin red tras reiniciar

```bash
rc-status                        # ¿está NetworkManager en ejecución?
rc-update show | grep -i network # ¿está en el nivel default?
ip -brief link                   # ¿reconoce el núcleo la interfaz?
```

Si la interfaz no aparece, falta su controlador en el núcleo. Identifica el
dispositivo y el módulo que necesita:

```bash
lspci -k | grep -A3 -i net
```

Configura la conexión con `nmtui`.

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/remedia.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **Español** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/troubleshooting.md)**

</div>
