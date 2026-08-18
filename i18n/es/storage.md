<!--
i18n/es/storage.md
@fraxgut
CC-BY-SA-4.0
Storage layout: partitioning, LUKS2, Btrfs subvolumes and the swapfile
-->

# Arquitectura de almacenamiento

> 🌐 **Idioma:** [English](../en/storage.md) · **Español**

Este documento describe el diseño de almacenamiento de la guía y las
decisiones que lo sustentan. El procedimiento completo de instalación está en
[installation.md](installation.md).

## Índice

1. [Las capas](#las-capas)
2. [Preparación del disco](#preparación-del-disco)
3. [Particionado](#particionado)
4. [Cifrado con LUKS2](#cifrado-con-luks2)
5. [Btrfs y subvolúmenes](#btrfs-y-subvolúmenes)
6. [Opciones de montaje](#opciones-de-montaje)
7. [Compresión](#compresión)
8. [TRIM y discard](#trim-y-discard)
9. [Montaje del sistema](#montaje-del-sistema)
10. [Espacio de intercambio](#espacio-de-intercambio)
11. [fstab](#fstab)
12. [Desbloqueo durante el arranque](#desbloqueo-durante-el-arranque)
13. [Variante con LVM](#variante-con-lvm)

## Las capas

La instalación por defecto usa tres capas:

```
Disco físico
    │
    ├── GPT
    │
    ├── partición 1 ── ESP (FAT32)  o  BIOS boot + /boot
    │
    └── partición 2 ── LUKS2 (Argon2id)
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

`/boot` queda **fuera** del contenedor cifrado. Esta decisión tiene una
consecuencia importante y deliberada: GRUB lee el núcleo y el initramfs de una
partición en claro, de modo que nunca necesita abrir LUKS. Argon2id se procesa
una sola vez, dentro del initramfs, donde `cryptsetup` dispone de la memoria y
del tiempo de CPU que la función requiere.

El sistema pide así la frase de paso una sola vez, tras el menú de GRUB. Un
diseño que cifre también `/boot` habilita `GRUB_ENABLE_CRYPTODISK` y la pide
dos veces: GRUB para poder leer `/boot`, y el initramfs de nuevo para el
dispositivo raíz. Esta guía conserva la petición única.

### La cadena de arranque

LUKS entrega confidencialidad a todo lo que vive dentro del contenedor. GRUB,
el núcleo y el initramfs quedan fuera, porque la máquina debe leerlos antes de
disponer de la frase de paso.

Eso separa dos propiedades de seguridad distintas. Quien se lleve el disco no
lee ninguno de tus datos. Quien tenga acceso físico repetido puede modificar el
núcleo o el initramfs, y un initramfs modificado captura la frase de paso en el
arranque siguiente.

Cifrar `/boot` responde a eso sólo en parte, porque el propio GRUB sigue siendo
legible y modificable. La respuesta completa verifica una firma sobre lo que la
máquina arranca:

```
Secure Boot  →  verificación de firma  →  GRUB, núcleo, initramfs
                                                ↓
                                          LUKS2 + Argon2id
```

Con los componentes de arranque firmados, la guía puede mantener `/boot` en
claro y conservar la petición única de la frase de paso. Ese trabajo está en la
hoja de ruta.

### Por qué Btrfs directamente sobre LUKS2

Btrfs ya entrega un único conjunto de almacenamiento con subvolúmenes,
instantáneas, cuotas y soporte para varios dispositivos. Un grupo de volúmenes
LVM cuyo único volumen lógico ocupa el 100 % del espacio y contiene un solo
sistema de archivos Btrfs añade una capa que no administra nada: cada
redimensionamiento, cada instantánea y cada reserva de espacio se resuelven
dentro de Btrfs.

LVM recupera su utilidad cuando el diseño necesita varios dispositivos de
bloque independientes bajo el mismo contenedor cifrado: un volumen para
máquinas virtuales, otro con un sistema de archivos distinto, o espacio del
grupo reservado sin asignar. Ese caso está documentado en
[Variante con LVM](#variante-con-lvm).

## Preparación del disco

Identifica el disco de destino y comprueba dos veces que sea el correcto. Los
comandos de esta sección destruyen todos los datos que contenga.

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

Define la variable con el disco elegido:

```bash
export DRIVE=/dev/nvme0n1
```

Sobrescribir el disco con datos aleatorios antes de crear el contenedor impide
distinguir después qué sectores contienen datos cifrados y cuáles nunca se
escribieron. El paso es opcional y tarda tanto como una escritura completa del
disco:

```bash
cryptsetup open --type plain --key-file /dev/urandom $DRIVE limpieza
dd if=/dev/zero of=/dev/mapper/limpieza status=progress bs=16M
cryptsetup close limpieza
```

Escribir ceros a través de un mapeo cifrado en modo plano produce datos
indistinguibles del azar y resulta bastante más rápido que leer de
`/dev/urandom` directamente.

Elimina después cualquier tabla de particiones previa:

```bash
sgdisk --zap-all $DRIVE
```

## Particionado

Determina primero si el sistema arranca por UEFI o por BIOS:

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

Elige **uno** de los dos esquemas.

### UEFI

Una partición de sistema EFI y el contenedor cifrado:

```bash
export DISK_LABEL="gentoosys"

sgdisk --clear \
       --new=1:0:+1GiB --typecode=1:ef00 --change-name=1:EFI \
       --new=2:0:0     --typecode=2:8309 --change-name=2:${DISK_LABEL} \
       $DRIVE

export EFI_PART_LABEL=EFI
export LUKS_PART_LABEL=${DISK_LABEL}
```

El código de tipo `8309` identifica una partición LUKS. `8300` también
funciona, pero `8309` describe el contenido con precisión y permite que las
herramientas del sistema reconozcan la partición como cifrada.

Un gibibyte para la ESP acomoda varios núcleos con sus initramfs
correspondientes, incluidos los generados durante las pruebas de arranque.

### BIOS

Tres particiones: el área que GRUB necesita para su segunda etapa, un `/boot`
separado y el contenedor cifrado.

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

## Cifrado con LUKS2

`cryptsetup` usa Argon2id como función de derivación por defecto desde la
versión 2.4.0. La RFC 9106 trata Argon2id como la variante principal, porque
combina la resistencia frente a ataques de canal lateral de Argon2i con la
resistencia frente a compromisos entre tiempo y memoria de Argon2d.

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

`--iter-time 5000` concede cinco segundos a la derivación de la clave.
`cryptsetup` calibra con ese presupuesto la memoria y las iteraciones que el
equipo tolera, de modo que el coste de un ataque por fuerza bruta se ajusta al
hardware disponible en lugar de a una constante fija.

La frase de paso protege todo lo demás. Elige una que resista un ataque de
diccionario: varias palabras aleatorias superan a una cadena corta con
sustituciones.

Abre el contenedor:

```bash
cryptsetup open /dev/disk/by-partlabel/${LUKS_PART_LABEL} cryptroot
```

El dispositivo descifrado queda en `/dev/mapper/cryptroot`.

> Si la sesión se interrumpe y vuelves a empezar, reabre el contenedor con este
> mismo comando antes de continuar.

## Btrfs y subvolúmenes

```bash
export BTRFS_LABEL="gentoobtrfs"
mkfs.btrfs --label ${BTRFS_LABEL} /dev/mapper/cryptroot
```

Monta la raíz del sistema de archivos para crear la estructura:

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

Cada subvolumen responde a un motivo concreto:

| Subvolumen   | Punto de montaje   | Motivo                                            |
|--------------|--------------------|---------------------------------------------------|
| `@`          | `/`                | La raíz del sistema instalado.                    |
| `@home`      | `/home`            | Conserva los datos personales al reinstalar.      |
| `@snapshots` | `/.snapshots`      | Aloja las instantáneas.                           |
| `@varlog`    | `/var/log`         | Mantiene los registros fuera de las instantáneas. |
| `@vartmp`    | `/var/tmp`         | Aísla los temporales.                             |
| `@portage`   | `/var/tmp/portage` | Recibe opciones de montaje propias.               |
| `@swap`      | `/var/swap`        | Contiene el archivo de intercambio.               |

`@portage` merece una explicación. Portage escribe y borra decenas de
gibibytes durante una compilación grande, y esos archivos desaparecen minutos
después. Un subvolumen propio permite montarlo con una compresión más barata,
o sin compresión, mientras el resto del sistema conserva la configuración
agresiva.

## Opciones de montaje

La configuración por defecto de la guía:

```
noatime,compress-force=zstd:5,space_cache=v2,nodiscard,subvol=@
```

Y para `@portage`:

```
noatime,compress=zstd:1,space_cache=v2,nodiscard,subvol=@portage
```

Tres observaciones sobre lo que **no** aparece:

**Btrfs detecta por sí mismo el tipo de dispositivo.** Consulta la topología
del dispositivo de bloque y determina si es rotacional, de modo que la guía le
deja ese trabajo. Fuerza `ssd` a mano únicamente si una capa virtual expone la
propiedad de forma incorrecta. Las antiguas optimizaciones de disposición
específicas para SSD se retiraron porque en el hardware actual no producían
beneficio y podían aumentar la fragmentación.

**La guía escribe las opciones que quiere.** `defaults` agrupa opciones que ya
están activas, así que la lista se mantiene corta y cada entrada significa
algo.

**`space_cache=v2` va explícito,** aunque hoy sea el comportamiento por
defecto. La guía declara la configuración deseada, y esa declaración se
mantiene estable aunque cambien los valores por defecto del núcleo. El único
coste es de compatibilidad: montar en escritura exige un núcleo con soporte
para el árbol de espacio libre, presente en cualquier núcleo moderno.

## Compresión

`compress-force` obliga a Btrfs a intentar comprimir cada extensión. Cuando el
resultado comprimido resultaría mayor que el original, Btrfs almacena los datos
sin comprimir, de modo que ningún archivo crece; lo que se gasta son ciclos de
CPU en un intento fallido.

La documentación de Btrfs recomienda `compress` a secas para uso general,
porque las heurísticas actuales aciertan bastante. La guía elige
`compress-force` por dos razones. La primera es la cobertura: un archivo puede empezar con datos
incompresibles y continuar con datos muy compresibles, y sin `force` una
decisión temprana puede excluir todo lo que viene después. La segunda es
declarativa: preferimos una política explícita sobre una heurística que cambia
entre versiones del núcleo.

### Elegir el nivel

Btrfs agrupa los niveles de zstd en tres tramos: de 1 a 3 casi en tiempo real,
de 4 a 8 más lentos con mejor razón de compresión, y de 9 a 15 con un esfuerzo
notablemente mayor cuya ganancia de tamaño puede ser pequeña.

El detalle que decide la elección es asimétrico: **el coste de comprimir crece
con el nivel, mientras que el de descomprimir se mantiene prácticamente
constante**. Los niveles altos penalizan la escritura y dejan la lectura casi
intacta.

| Nivel     | Rol en la guía        | Cuándo conviene                                                                |
|-----------|-----------------------|--------------------------------------------------------------------------------|
| `zstd:5`  | Predeterminado        | Estación de trabajo moderna: buen equilibrio entre espacio y CPU.              |
| `zstd:9`  | Compresión alta       | CPU abundante y almacenamiento más valioso que el tiempo de escritura.         |
| `zstd:15` | Compresión máxima     | Datos poco mutables que se escriben una vez y se leen muchas: se lee más de lo que se escribe. |

`zstd:5` es el valor predeterminado de la guía. Comprime de forma más decidida
que el valor por defecto de Btrfs y conserva un rendimiento de escritura
razonable en hardware actual.

`zstd:9` y `zstd:15` son opciones plenamente válidas, y el criterio que las
justifica es la carga de trabajo, no la potencia del equipo. Un procesador de
muchos núcleos no convierte `zstd:15` en la mejor opción para
`/var/tmp/portage`: allí se gastaría CPU comprimiendo archivos que se borrarán
en minutos, en competencia directa con la compilación. En un conjunto de datos
estable que permanece meses en disco, la ecuación se invierte.

Por eso `@portage` usa `compress=zstd:1`: la compresión más barata disponible,
sin `force`, para que Portage y Clang dispongan de los núcleos.

## TRIM y discard

TRIM informa al dispositivo de qué bloques dejaron de contener datos útiles.
Sin esa información, el controlador del SSD trabaja como si tuviera mucho menos
espacio libre del que realmente existe, lo que degrada la recolección de basura
y el rendimiento sostenido.

La contrapartida es de privacidad. El núcleo advierte que propagar TRIM a
través de dm-crypt revela información sobre el volumen: qué regiones están
libres, cuánto espacio se usa y algunas características del sistema de
archivos. La clave, los nombres de archivo y su contenido siguen protegidos;
lo que se filtra es el patrón de asignación.

Como TRIM debe atravesar todas las capas, la decisión se toma en dos lugares:
en Btrfs y en dm-crypt.

| Perfil            | dm-crypt         | Btrfs            | Prioriza                          |
|-------------------|------------------|------------------|-----------------------------------|
| **A** Máxima privacidad | sin discard | `nodiscard`      | Opacidad del patrón de asignación |
| **B** Equilibrado | `allow-discards` | `nodiscard` + `fstrim` semanal | Privacidad temporal y salud del SSD |
| **C** Rendimiento | `allow-discards` | `discard=async`  | Consistencia sostenida del SSD    |

**La guía usa el perfil B.**

El perfil A conserva la máxima opacidad, pero renuncia por completo a que el
SSD conozca su espacio libre. En una máquina Gentoo el coste se acumula: cada
compilación grande escribe y descarta decenas de gibibytes, y el controlador
termina administrando un disco que cree mucho más lleno de lo que está.

El perfil C entrega al dispositivo una imagen casi continua del espacio
liberado. Btrfs agrupa las extensiones y limita las operaciones, de modo que su
impacto en el rendimiento es pequeño. A cambio, un observador con acceso al
almacenamiento obtiene información actualizada de qué se liberó y cuándo.

El perfil B ejecuta el TRIM por lotes cuando el equipo está ocioso. Obtiene
casi toda la ventaja práctica de C —la página de manual de `fstrim` considera
suficiente una ejecución semanal para un escritorio o servidor normal— y
elimina la resolución temporal: en lugar de anunciar cada liberación en el
momento en que ocurre, entrega un conjunto grande una vez por semana. El patrón
de espacio libre acaba siendo visible de todos modos; lo que se pierde es la
línea de tiempo.

`allow-discards` se activa en la línea de órdenes del núcleo, junto al resto de
la configuración de dracut:

```
rd.luks.allow-discards
```

Y el recorte periódico se programa con cron:

```bash
emerge sys-process/cronie
rc-update add cronie default

cat > /etc/cron.weekly/fstrim <<'EOF'
#!/bin/sh
# Releases unused blocks to the device once a week. The batch runs while
# the machine is idle, which keeps the discard requests large and away
# from interactive work.
exec /usr/sbin/fstrim --all --quiet
EOF
chmod +x /etc/cron.weekly/fstrim
```

Para adoptar el perfil C, reemplaza `nodiscard` por `discard=async` en las
opciones de montaje y omite la tarea de cron. Para el perfil A, retira
`rd.luks.allow-discards` de la línea del núcleo.

## Montaje del sistema

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

Monta después la partición de arranque.

Para UEFI:

```bash
mkfs.vfat -F 32 -n EFI /dev/disk/by-partlabel/${EFI_PART_LABEL}
mount -o rw,nosuid,nodev,noexec,relatime,fmask=0077,dmask=0077 \
      /dev/disk/by-partlabel/${EFI_PART_LABEL} /mnt/gentoo/boot
```

Las máscaras `0077` restringen la partición a `root`. La ESP no admite
permisos POSIX, de modo que la restricción se aplica en el montaje: sin ella,
cualquier usuario podría leer el núcleo y el initramfs.

Para BIOS:

```bash
mkfs.ext4 -L BOOT /dev/disk/by-partlabel/${BOOT_PART_LABEL}
mount -o rw,nosuid,nodev,relatime \
      /dev/disk/by-partlabel/${BOOT_PART_LABEL} /mnt/gentoo/boot
```

## Espacio de intercambio

Btrfs incorpora desde la versión 6.1 de `btrfs-progs` una orden que crea el
archivo de intercambio con los atributos correctos —sin copia en escritura, sin
compresión— y ejecuta `mkswap` sobre él:

```bash
btrfs filesystem mkswapfile --size 8G /mnt/gentoo/var/swap/swapfile
swapon /mnt/gentoo/var/swap/swapfile
```

Esa única orden reemplaza la secuencia de `truncate`, `chattr +C`, `dd` y
`mkswap` que antes hacía falta.

Como referencia de tamaño:

| RAM    | Sin hibernación | Con hibernación |
|--------|-----------------|-----------------|
| 4 GiB  | 2 GiB           | 6 GiB           |
| 8 GiB  | 3 GiB           | 11 GiB          |
| 16 GiB | 4 GiB           | 20 GiB          |
| 32 GiB | 6 GiB           | 38 GiB          |
| 64 GiB | 8 GiB           | 72 GiB          |

### Hibernación

La hibernación necesita el desplazamiento físico del archivo dentro del sistema
de archivos. Ese valor **no** es el que informa `filefrag`; `btrfs-progs` lo
calcula:

```bash
btrfs inspect-internal map-swapfile -r /mnt/gentoo/var/swap/swapfile
```

El número resultante se pasa al núcleo como `resume_offset`, junto con
`resume=` apuntando al dispositivo descifrado. La configuración completa está
en [installation.md](installation.md).

## fstab

El archivo de intercambio se referencia **por ruta**. El UUID que produce
`mkswap` identifica el «sistema de archivos» de intercambio y, al residir
dentro de un archivo, no es visible ni utilizable como identificador de
dispositivo. Una entrada `UUID=` para un archivo de intercambio no funciona.

```bash
export ROOT_UUID=$(blkid -s UUID -o value /dev/mapper/cryptroot)
export BOOT_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${EFI_PART_LABEL})
```

```
# <sistema de archivos>  <punto de montaje>  <tipo>  <opciones>                                                      <dump> <pass>

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

La sexta columna vale `0` en todas las entradas Btrfs: `fsck` no tiene nada que
hacer sobre Btrfs, que verifica sus sumas de comprobación al leer.

## Desbloqueo durante el arranque

El initramfs abre el contenedor LUKS antes de montar la raíz, guiado por los
parámetros del núcleo que genera dracut. Con esa configuración, el servicio
`dmcrypt` de OpenRC no interviene en la raíz.

`dmcrypt` sólo hace falta para volúmenes cifrados adicionales que se abren
después del arranque. En OpenRC se configura en `/etc/conf.d/dmcrypt`, con una
sintaxis propia que **no** es la de `/etc/crypttab`:

```bash
# /etc/conf.d/dmcrypt
target=datos
source='UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
```

```bash
rc-update add dmcrypt boot
```

## Variante con LVM

LVM aporta valor cuando el contenedor cifrado debe alojar varios dispositivos
de bloque independientes. El grupo de volúmenes se crea sobre el dispositivo
descifrado:

```
GPT
 └── LUKS2
       └── LVM2 (grupo de volúmenes)
             ├── lv_root  → Btrfs   →  /
             ├── lv_vm    → XFS     →  imágenes de máquinas virtuales
             └── espacio libre reservado
```

```bash
pvcreate /dev/mapper/cryptroot
vgcreate vg_gentoo /dev/mapper/cryptroot

lvcreate --size 200G --name lv_root vg_gentoo
lvcreate --size 300G --name lv_vm   vg_gentoo

mkfs.btrfs --label gentoobtrfs /dev/vg_gentoo/lv_root
mkfs.xfs -L vmstore /dev/vg_gentoo/lv_vm
```

Deja espacio sin asignar en el grupo si prevés ampliar algún volumen más
adelante: ésa es justamente la flexibilidad que LVM entrega y Btrfs no.

Con esta variante, el sistema de archivos raíz deja de estar en
`/dev/mapper/cryptroot` y pasa a `/dev/vg_gentoo/lv_root`. Ajusta en
consecuencia los montajes, el `fstab` y la línea de órdenes del núcleo, que
además necesita el parámetro correspondiente:

```
rd.lvm.lv=vg_gentoo/lv_root
```

El initramfs debe incluir el módulo `lvm`, y el servicio `lvm` de OpenRC debe
estar en el nivel `boot`:

```bash
rc-update add lvm boot
```

Si en cambio el diseño se reduce a un volumen lógico que ocupa todo el grupo,
LVM no administra nada: usa el esquema por defecto de esta guía.

---

> 🌐 **Idioma:** [English](../en/storage.md) · **Español**
