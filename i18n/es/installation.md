<!--
i18n/es/installation.md
@fraxgut
CC-BY-SA-4.0
The installation procedure from the live medium to the first boot
-->

# Instalación

> 🌐 **Idioma:** [English](../en/installation.md) · **Español**

## Prerequisitos

### Conocimientos Previos

*   Dominio de la terminal Linux.
*   Comprensión de particionado de discos (MBR/GPT), sistemas de archivos (ext4, VFAT, BTRFS).
*   Conocimiento básico de LVM (Logical Volume Management) y LUKS (Linux Unified Key Setup).
*   Experiencia previa con Gentoo: compilación de paquetes (`emerge`), gestión de USE flags, `make.conf`, perfiles (`eselect profile`).
*   Experiencia (o disposición a investigar extensamente) en la configuración y compilación manual del kernel Linux.

### Descargar un Sistema para la Instalación

Necesitas un entorno Linux funcional para realizar la instalación. La imagen oficial de Gentoo es válida, pero esta guía recomienda **SystemRescueCD** por su conjunto de herramientas.

*   **Descarga:** [https://www.system-rescue.org/Download/](https://www.system-rescue.org/Download/)

### Definir el Medio de Instalación

Elige cómo arrancarás SystemRescueCD en la máquina destino (llamada **sistema invitado**):

1.  **Dispositivo Físico (USB/CD/DVD):** Para instalar en hardware real.
2.  **Máquina Virtual (VirtualBox, QEMU, VMWare):** Montando la ISO directamente.
3.  **Conexión Remota (VPS):** Tu proveedor montará la ISO por ti o te dará acceso a una consola VNC/IPMI.

El equipo desde el que preparas el medio es el **sistema anfitrión**.

### Grabar la Imagen (Caso 1)

Se recomienda USB para GPT/UEFI en sistemas UNIX-like (GNU/Linux, macOS, BSD):

1.  Descarga la ISO y (opcionalmente) renómbrala a `SystemRescueCD.iso`.
2.  Identifica tu dispositivo USB (¡con cuidado!): `lsblk` (ej: `/dev/sdc`). **Asegúrate de elegir el correcto, ¡los datos se borrarán!**
3.  Define la variable (reemplaza `sdc`): `export USB_DEV=/dev/sdc`
4.  **¡Borrado completo del USB!**
    ```bash
    sgdisk --zap-all ${USB_DEV}
    sgdisk --clear --new 1:0:0 --typecode=1:ef00 --change-name=1:LiveUSB ${USB_DEV} # Para UEFI
    # O adapta para BIOS si es necesario
    mkfs.fat -F 32 -n LiveUSB ${USB_DEV}1 # Asumiendo que la partición es la 1
    ```
5.  Graba la ISO (puede requerir `isohybrid` si la ISO no lo es):
    ```bash
    # Opcional si la ISO no es híbrida: isohybrid SystemRescueCD.iso
    dd if=SystemRescueCD.iso of=${USB_DEV} bs=4M status=progress conv=fsync
    ```
6.  Expulsa el USB de forma segura.

*   **Windows:** Usa [Rufus](https://rufus.ie/es/) o una herramienta similar.
*   **CD/DVD:** Usa las herramientas de grabación de tu sistema.

Conecta el medio al **sistema invitado** y arranca desde él.

### Máquina Virtual (Caso 2)

Descarga la ISO y configúrala como unidad de CD/DVD virtual en tu software de virtualización. Arranca la VM desde la ISO.

### Conexión Remota (Caso 3)

Contacta a tu proveedor para que monte la ISO de SystemRescueCD en tu servidor virtual. Accede a través de la consola proporcionada (SSH no estará disponible hasta que lo configures en el entorno live).

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- INSTALACIÓN -->
## Instalación

Sigue estos pasos cuidadosamente. Reemplaza las variables y nombres de ejemplo (`NOMBRE_xxx_sys`, `NOMBRE_vg1`, `nombreusuario`, etc.) con tus propias elecciones.

### Nota sobre los Marcadores `[x]`

Verás marcadores como `[x]` al final de algunas líneas de comandos. Estos indican **puntos de control importantes** o lugares donde podrías necesitar **reintroducir variables o comandos si tu sesión de instalación (ej. SSH) se interrumpe y tienes que volver a entrar al entorno `chroot` o retomar desde un punto específico.** Presta atención a las variables definidas antes de estos marcadores.

### Paso 1: Preparación del Entorno de Instalación

Una vez arrancado SystemRescueCD:

1.  **Conexión a Internet:**
    *   SystemRescueCD usa NetworkManager. Si usas cable, la conexión puede ser automática.
    *   Para configurar (especialmente Wi-Fi o IPs estáticas), usa la interfaz de texto: `nmtui`
    *   Verifica la conexión: `ping -c 3 9.9.9.9`
2.  **Contraseña de Root (Temporal):** Establece una contraseña para el usuario `root` del entorno live para seguridad y para SSH:
    ```bash
    passwd
    # Elige una contraseña segura pero temporal
    ```
3.  **(Opcional) Configurar SSH para Acceso Remoto:** Útil para copiar/pegar comandos desde tu máquina anfitriona.
    *   Edita la configuración: `vim /etc/ssh/sshd_config` (o usa `nano`)
    *   Descomenta/Ajusta:
        *   `Port 22`
        *   `ListenAddress 0.0.0.0` (o la IP específica de la máquina invitada)
        *   `PasswordAuthentication yes` (solo para la instalación, considera desactivarlo después)
        *   `PermitRootLogin yes` (solo para la instalación)
    *   Obtén la IP: `ip addr`
    *   Inicia el servicio SSH: `systemctl start sshd`
    *   **Nota sobre Firewall:** SystemRescueCD puede tener `iptables` activo. Para simplificar, puedes detenerlo: `systemctl stop iptables`. Si prefieres mantenerlo, necesitas abrir el puerto 22 (TCP).
    *   Desde tu máquina anfitriona, conecta: `ssh root@<IP_DEL_SISTEMA_INVITADO>`
4.  **Sincronizar Reloj:** Es crucial para la verificación de paquetes y certificados.
    ```bash
    ntpd -q -g
    ```

### Paso 3: Instalación del Sistema Base Gentoo

#### Descarga y Verificación del Stage3

1.  Busca el enlace al Stage3 más reciente para `amd64-musl-hardened` en [Gentoo Downloads](https://www.gentoo.org/downloads/).
    *   **¡Actualiza este enlace!** El que sigue es solo un ejemplo y probablemente esté desactualizado.
    ```bash
    # EJEMPLO DESACTUALIZADO - BUSCA EL NUEVO EN gentoo.org/downloads/
    export STAGE3_URL="https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20230409T163155Z/stage3-amd64-musl-hardened-20230409T163155Z.tar.xz"
    export STAGE3_FILE=$(basename ${STAGE3_URL})

    wget ${STAGE3_URL}
    wget ${STAGE3_URL}.CONTENTS.gz # Opcional pero útil
    wget ${STAGE3_URL}.DIGESTS.asc # Para verificar firmas
    wget ${STAGE3_URL}.asc # Firma del archivo principal
    ```
2.  **Verifica la Integridad y Autenticidad:**
    *   Importa las claves de Gentoo Release Engineering:
        ```bash
        wget -O - https://qa-reports.gentoo.org/output/service-keys.gpg | gpg --import
        # Puede que necesites instalar gnupg si no está: emerge app-crypt/gnupg
        ```
    *   Verifica la firma del archivo DIGESTS:
        ```bash
        gpg --verify ${STAGE3_FILE}.DIGESTS.asc
        # Busca "Good signature from" y una clave de Gentoo Release Engineering.
        ```
    *   Verifica los hashes SHA512 y Whirlpool del archivo stage3 contra los del archivo DIGESTS:
        ```bash
        grep -A2 ${STAGE3_FILE} ${STAGE3_FILE}.DIGESTS.asc
        sha512sum ${STAGE3_FILE}
        whirlpooldeep ${STAGE3_FILE} # Puede requerir instalar 'whirlpooldeep' o usar openssl
        # openssl dgst -r -sha512 ${STAGE3_FILE}
        # openssl dgst -r -whirlpool ${STAGE3_FILE}
        # Compara los hashes visualmente.
        ```

#### Extracción del Stage3 y Configuración de Portage

1.  Extrae el Stage3 (preservando atributos y permisos):
    ```bash
    tar xpvf ${STAGE3_FILE} --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
    ```
2.  Limpia el archivo descargado:
    ```bash
    # rm ${STAGE3_FILE} ${STAGE3_FILE}.DIGESTS.asc ${STAGE3_FILE}.asc ${STAGE3_FILE}.CONTENTS.gz
    ```
3.  Configura el archivo `repos.conf` de Portage:
    ```bash
    mkdir -p /mnt/gentoo/etc/portage/repos.conf
    cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
    ```
4.  Copia la configuración DNS del sistema live (NetworkManager la gestiona):
    ```bash
    cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
    ```

#### Montajes Adicionales y Entrada al Chroot

Montamos sistemas de archivos virtuales necesarios para el entorno `chroot`.

```bash
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys # Importante para systemd/udev si se usara
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev # Importante

# Montaje tmpfs para /dev/shm si no existe o es un enlace roto
test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
mount -t tmpfs -o nosuid,nodev,noexec shm /dev/shm
chmod 1777 /dev/shm

# Opcional: Montar /run si es necesario (normalmente systemd lo gestiona, pero puede ser útil)
# mount --bind /run /mnt/gentoo/run
# mount --make-slave /mnt/gentoo/run
```

*Marcador:* `[x]` Montajes virtuales listos para chroot.

#### Configuración Inicial dentro del Chroot

1.  Entra al entorno `chroot`:
    ```bash
    chroot /mnt/gentoo /bin/bash
    source /etc/profile
    export PS1="(chroot) ${PS1}" # Cambia el prompt para saber que estás en chroot
    ```
    *Marcador:* `[x]` **¡Estás dentro del chroot!** Si sales y vuelves a entrar, necesitas re-exportar `PS1` y posiblemente otras variables de entorno definidas anteriormente (`DRIVE`, `LUKS_UUID`, etc.).
2.  **Importante:** Vuelve a montar `/boot` si es necesario (especialmente si saliste y entraste de nuevo). Verifica con `findmnt /boot`. Si no está montado:
    *   `mount /dev/disk/by-partlabel/${EFI_PART_LABEL} /boot` (UEFI)
    *   `mount /dev/disk/by-partlabel/${BOOT_PART_LABEL} /boot` (BIOS)

#### Configuración de Portage (make.conf y Repositorios)

1.  **Sincronizar Portage por primera vez:** Usa `emerge-webrsync` para obtener una instantánea del árbol de Portage y metadatos.
    ```bash
    emerge-webrsync
    ```
2.  **Seleccionar el Perfil Correcto:** Asegúrate de que el perfil `hardened/linux/musl/amd64` (o similar) esté activo.
    ```bash
    eselect profile list
    # Busca el número del perfil hardened/linux/musl/amd64 (puede variar ligeramente)
    # export PROFILE_NUM=... # Reemplaza ... con el número
    # eselect profile set --force ${PROFILE_NUM}
    ```
3.  **Configurar `make.conf`:** Este es el archivo central de configuración de Portage. Edítalo con `nano` o `vim`.
    ```bash
    nano /etc/portage/make.conf
    ```
    Añade/Modifica las siguientes líneas (adapta `NPROC` y `CPU_FLAGS_X86` a tu hardware):

    ```ini
    # /etc/portage/make.conf

    # --- Variables Básicas de Compilación ---
    # Ajusta '-march=native' si tienes problemas o compilas para otra máquina.
    # O2 es un buen balance, O3 puede ser más rápido pero a veces inestable.
    COMMON_FLAGS="-march=native -O3 -pipe"
    CFLAGS="${COMMON_FLAGS}"
    CXXFLAGS="${COMMON_FLAGS}"
    FCFLAGS="${COMMON_FLAGS}" # Fortran
    FFLAGS="${COMMON_FLAGS}"  # Fortran

    # Arquitectura y Host (ya debería estar correcto por el stage3)
    CHOST="x86_64-gentoo-linux-musl"

    # --- Opciones de Paralelización ---
    # NPROC = Número de hilos de CPU. 'nproc' lo detecta.
    NPROC=$(nproc) # Ejecutar $(nproc) en CLI y reemplazar con el valor obtenido
    MAKEOPTS="-j${NPROC} -l${NPROC}" # -l para limitar carga media
    EMERGE_DEFAULT_OPTS="--jobs ${NPROC} --load-average ${NPROC}"

    # --- Opciones de Prioridad (Opcional pero recomendado) ---
    # Baja la prioridad de CPU e I/O de emerge para no congelar el sistema
    # PORTAGE_NICENESS="19"
    # PORTAGE_IONICE_CLASS="3" # Idle

    # --- Configuración Regional (se ajustará mejor después) ---
    L10N="es-CL es" # Añade tus locales deseadas separadas por espacio
    LINGUAS="es_CL es" # Para paquetes que usan LINGUAS

    # --- USE Flags Globales ---
    # Aquí defines características globales. Es CRUCIAL entenderlas.
    # (-) deshabilita, (+) habilita (implícito si no hay signo).
    # Esta es una base para este setup específico. ¡INVESTIGA Y ADAPTA!
    # Quita cosas que no usarás (ej. -X si es servidor, -gnome -kde, etc.)
    # Añade soporte para lo que SÍ usarás (btrfs, lvm, luks ya debería estar)
    USE="btrfs lvm cryptsetup hardened musl X -gnome -kde -systemd readline ncurses unicode cli vim-syntax zsh-completion"
    # Agrega 'llvm clang lto' si planeas seguir toda la guía.
    # Agrega 'cpu_flags_x86_...' detectadas abajo.

    # --- Aceptación de Licencias ---
    # Permite solo licencias libres (@FREE) y explícitamente aceptadas.
    ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE" # @BINARY para firmware, etc.

    # --- Aceptación de Keywords (Estable vs Inestable) ---
    # "" -> Estable (recomendado para empezar)
    # "~amd64" -> Inestable (testing, más actual pero potencialmente con bugs)
    ACCEPT_KEYWORDS="~amd64" # Cambia a "" si prefieres estabilidad

    # --- CPU Flags Específicas (Auto-detección) ---
    # Instala cpuid2cpuflags y añade las flags detectadas a USE
    # emerge app-portage/cpuid2cpuflags
    # echo 'CPU_FLAGS_X86="'$(cpuid2cpuflags)'"' >> /etc/portage/make.conf
    # O añade manualmente las flags de la salida de cpuid2cpuflags a la variable USE.

    # --- Opciones de Hardware (Ejemplos) ---
    VIDEO_CARDS="intel i965" # Ejemplo para Intel, usa 'nvidia', 'radeon', 'amdgpu', 'vmware', 'qxl', etc.
    INPUT_DEVICES="libinput synaptics" # Ejemplo para touchpad/teclado

    # --- Selección de Mirrors ---
    # Descomenta y edita GENTOO_MIRRORS o usa 'mirrorselect' después.
    # GENTOO_MIRRORS="https://mirror.ejemplo.com/gentoo/"

    # --- Opciones de GRUB ---
    # Especifica las plataformas para las que GRUB debe compilarse
    GRUB_PLATFORMS="efi-64" # Para UEFI
    # GRUB_PLATFORMS="pc" # Para BIOS

    # --- MUSL específico (puede que no sea necesario si el perfil lo gestiona) ---
    # C_INCLUDE_PATH="/usr/include/musl"
    ```
4.  **Instalar `cpuid2cpuflags` y Configurar Mirrors:**
    ```bash
    emerge app-portage/cpuid2cpuflags app-portage/mirrorselect
    # Detectar CPU flags y añadirlas a make.conf (EDITA el archivo y pega la línea):
    cpuid2cpuflags
    echo "Añade la línea 'CPU_FLAGS_X86=...' de arriba a /etc/portage/make.conf en la variable USE o como variable separada."
    # Seleccionar mirrors (interactivo):
    mirrorselect -i -o >> /etc/portage/make.conf
    # Revisa /etc/portage/make.conf para asegurar que todo esté bien.
    nano /etc/portage/make.conf
    ```
5.  **Añadir Repositorios Adicionales (Overlays):** Necesitamos overlays para `musl-locales`, `gentooLTO`, `toolchain-clang`, etc. Usa `eselect repository`.
    ```bash
    emerge app-eselect/eselect-repository dev-vcs/git

    # Repositorio GURU (contiene muchas cosas útiles)
    eselect repository enable guru
    # Repositorio para musl-locales (del usuario 12101111)
    eselect repository add 12101111-overlay git https://github.com/12101111/overlay.git
    # Repositorios para LTO y LLVM/Clang (se añadirán más tarde si sigues esos pasos)

    # Sincroniza todos los repositorios
    emerge --sync
    ```
6.  **Actualizar el Sistema `@world`:** Aplica los cambios de perfil y `make.conf`. Esto puede tardar mucho.
    ```bash
    emerge -uvDN @world
    # -u: update, -v: verbose, -D: deep dependencies, -N: new use changes
    # Revisa los cambios propuestos antes de aceptar.
    ```
7.  **Limpiar Dependencias Obsoletas:**
    ```bash
    emerge --depclean
    ```
8.  **Herramientas Útiles:** Instala un editor decente si no lo tienes.
    ```bash
    emerge app-editors/neovim # O app-editors/vim, app-editors/nano
    ```

#### Configuración de MUSL (Localización y Zona Horaria)

MUSL requiere pasos adicionales para la configuración regional.

1.  **Instalar `musl-locales`:**
    *   Puede requerir aceptar keywords inestables para este paquete si no lo hiciste globalmente. Crea el archivo si no existe: `mkdir -p /etc/portage/package.accept_keywords`
    ```bash
    echo "sys-apps/musl-locales ~amd64" >> /etc/portage/package.accept_keywords/musl-locales
    emerge sys-apps/musl-locales
    ```
2.  **Configurar Zona Horaria:**
    *   Instala `timezone-data` y `ntp` (o `chrony`).
    ```bash
    emerge sys-libs/timezone-data net-misc/ntp
    ```
    *   Busca tu zona horaria: `ls /usr/share/zoneinfo` (ej: `America/Santiago`, `Europe/Madrid`)
    *   Establece la zona horaria (reemplaza `Country/City`):
    ```bash
    export TZ="Country/City"
    echo "TZ=\"${TZ}\"" > /etc/env.d/00musl # Guarda para futuras sesiones
    # Actualiza el entorno actual
    env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
    # Verifica
    date
    ```
3.  **Configurar Locales MUSL:**
    *   Define las variables de locale (ej. `es_CL.UTF-8`, `en_US.UTF-8`). Elige una como principal.
    ```bash
    export MUSL_LOCPATH="/usr/share/i18n/locales/musl"
    export CHARSET="UTF-8" # O el que uses
    export LANG="es_CL.${CHARSET}" # Tu locale principal
    export LC_COLLATE="C" # O tu locale principal, 'C' es más rápido para ordenar a veces

    # Guarda en /etc/env.d/00musl
    echo "MUSL_LOCPATH=\"${MUSL_LOCPATH}\"" >> /etc/env.d/00musl
    echo "CHARSET=\"${CHARSET}\"" >> /etc/env.d/00musl
    echo "LANG=\"${LANG}\"" >> /etc/env.d/00musl
    echo "LC_COLLATE=\"${LC_COLLATE}\"" >> /etc/env.d/00musl

    # Actualiza y verifica
    env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
    locale -a # Debería mostrar las locales generadas por musl-locales
    eselect locale list
    # eselect locale set <NUMERO_DE_TU_LOCALE>
    ```
4.  **Sincronizar Reloj de Hardware:**
    ```bash
    # Sincroniza con servidor NTP
    ntpd -q -g
    # Escribe la hora UTC al reloj de hardware
    hwclock --systohc --utc
    ```

### Paso 4: Compilación del Kernel

Esta es una de las partes más críticas y específicas de Gentoo.

#### Instalación de Fuentes y Firmware

1.  **Linux Firmware:** Necesario para muchos drivers (Wi-Fi, GPU, etc.). Es código binario no libre.
    *   Asegúrate de que `ACCEPT_LICENSE` en `make.conf` incluye `@BINARY-REDISTRIBUTABLE`.
    ```bash
    emerge sys-kernel/linux-firmware
    ```
2.  **Fuentes del Kernel (Zen):**
    *   Puede requerir aceptar keywords inestables.
    ```bash
    echo "sys-kernel/zen-sources ~amd64" >> /etc/portage/package.accept_keywords/zen-sources
    emerge sys-kernel/zen-sources
    ```
3.  **Seleccionar el Kernel:** `eselect kernel` gestiona el enlace simbólico `/usr/src/linux`.
    ```bash
    eselect kernel list
    # eselect kernel set <NUMERO_DEL_KERNEL_ZEN>
    cd /usr/src/linux
    ```
4.  **Herramienta Initramfs (Dracut):** Necesaria para cargar el módulo LUKS y LVM antes de montar la raíz.
    ```bash
    emerge sys-kernel/dracut
    ```

#### Configuración del Kernel (Zen)

Configurar manualmente el kernel es complejo. **Debes investigar tu hardware** (`lspci -k`, `lsusb`, `lscpu`) y habilitar los drivers necesarios.

**Enfoques:**

1.  **Manual (Recomendado para Optimizar):**
    *   Empieza desde una configuración base: `make defconfig` o copia una configuración existente.
    *   Usa `make menuconfig` para la configuración interactiva basada en texto.
    *   **Áreas Clave a Revisar (investiga cada opción):**
        *   `General setup`: Compresión del kernel (LZ4/ZSTD), Soporte Initramfs.
        *   `Processor type and features`: Tipo de CPU (AMD/Intel), SMP, Soporte EFI, Virtualización (KVM si la usarás).
        *   `Power management`: CPUFreq governors (ondemand/performance/schedutil).
        *   `Enable loadable module support`: Fundamental.
        *   `Block layer`: IO Schedulers (BFQ recomendado para escritorio), Soporte Particiones (GPT).
        *   `Device Drivers`:
            *   `Graphics support`: Drivers para tu GPU (Intel/AMD/Nvidia/VirtIO). ¡Crucial para entorno gráfico! Habilita KMS.
            *   `Network device support`: Drivers para tu tarjeta de red cableada y/o Wi-Fi.
            *   `Input device support`: Teclado, ratón, touchpad.
            *   `SCSI device support`, `Serial ATA/PATA`, `NVME Support`: Drivers para tus discos duros/SSD.
            *   `Multiple devices driver support (RAID and LVM)`: **FUNDAMENTAL habilitar `Device mapper support` y `Crypt target support`** para LUKS y LVM.
            *   `Virtio drivers` (si es VM).
        *   `File systems`: **FUNDAMENTAL habilitar `Btrfs filesystem support`**, `Ext4` (para /boot si usaste ext4), `VFAT` (para EFI), `Proc`, `Tmpfs`, `Devtmpfs`.
        *   `Cryptographic API`: Habilita los cifrados y hashes usados por LUKS (`AES`, `XTS`, `SHA512`, `Argon2`).
        *   `Library routines`: Asegúrate de que las rutinas de compresión/descompresión necesarias (LZ4/ZSTD si las usas) estén habilitadas.
    *   **¡Guarda tu configuración!** (`.config`)

2.  **Semi-Automático (`genkernel`):** Simplifica la creación del initramfs, pero puede generar un kernel más grande. No usado en esta guía.

3.  **Usar `.config` Existente:** Busca configuraciones de ejemplo para hardware similar o para Zen kernel.

**Recomendación para esta guía:** Usa `make menuconfig` y enfócate en habilitar **todo lo necesario para BTRFS, LVM, LUKS (crypt target), tu sistema de archivos /boot (VFAT o Ext4), tus discos (NVMe/SATA/SCSI), red, y gráficos.**

#### Compilación e Instalación del Kernel con Dracut

1.  **Configurar Dracut:** Edita `/etc/dracut.conf` para incluir los módulos necesarios en el initramfs.
    ```bash
    nano /etc/dracut.conf
    ```
    Asegúrate de que estas líneas (o similares) estén presentes y descomentadas:
    ```ini
    hostonly="yes" # Incluye solo módulos para el host actual
    compress="zstd" # O lz4, xz. Debe estar habilitado en el kernel.
    # Añade módulos cruciales para nuestro setup:
    add_dracutmodules+=" crypt lvm btrfs resume "
    # Añade también drivers de teclado USB si tu /boot está encriptado y necesitas teclear la pass
    # add_drivers+=" hid_generic usbhid "
    # Para UEFI:
    uefi="yes"
    ```
2.  **Compilar el Kernel:** (Esto puede tardar bastante)
    ```bash
    # -jNPROC acelera la compilación usando todos los núcleos
    make -j$(nproc)
    ```
3.  **Instalar Módulos:**
    ```bash
    make modules_install
    ```
4.  **Instalar el Kernel:** Copia la imagen del kernel a `/boot`.
    ```bash
    make install
    ```
5.  **Generar Initramfs con Dracut:**
    *   Obtén la versión exacta del kernel instalado: `basename /boot/vmlinuz-*` o mira la salida de `make install`.
    *   Reemplaza `KERNEL_VERSION` abajo con la versión correcta (ej. `5.15.88-zen1-gentoo`).
    ```bash
    KERNEL_VERSION=$(basename /boot/vmlinuz-*) # Intenta autodetectar
    # Verifica KERNEL_VERSION: echo ${KERNEL_VERSION}
    dracut --force --kver ${KERNEL_VERSION}
    # Esto creará /boot/initramfs-${KERNEL_VERSION}.img
    # Verifica que los módulos lvm, crypt, btrfs estén listados con: lsinitrd /boot/initramfs-${KERNEL_VERSION}.img | grep -E 'lvm|crypt|btrfs'
    ```

### Paso 5: Configuración Final del Sistema

#### Configuración de Red, Hostname y Usuarios

1.  **Hostname:** Elige un nombre para tu máquina.
    ```bash
    echo "MiGentooMusl" > /etc/hostname # Reemplaza MiGentooMusl
    ```
2.  **Contraseña de Root (Permanente):** Establece la contraseña final para el superusuario. ¡Hazla segura!
    ```bash
    passwd
    ```
3.  **Crear Usuario Regular:** No uses `root` para tareas diarias.
    ```bash
    # Reemplaza 'miusuario' con tu nombre de usuario
    # -m: crea home dir, -G: grupos (wheel para sudo/doas, audio, video, usb, etc.)
    useradd -m -G wheel,users,audio,video,usb,portage miusuario
    # Establece la contraseña para tu usuario
    passwd miusuario
    ```
4.  **Configurar Red (NetworkManager):**
    *   Asegúrate de que `networkmanager` esté en tus USE flags globales (`/etc/portage/make.conf`) o instálalo explícitamente. Necesita la USE flag `dbus`.
    ```bash
    echo "net-wireless/wpa_supplicant dbus" >> /etc/portage/package.use/wpa_supplicant # Necesario para Wi-Fi
    emerge net-misc/networkmanager
    # Habilita el servicio para que inicie al arrancar (asumiendo OpenRC)
    rc-update add NetworkManager default
    ```

#### Servicios Esenciales y Herramientas

1.  **Sistema de Logging (metalog):**
    ```bash
    emerge app-admin/metalog
    rc-update add metalog default
    ```
2.  **Cliente NTP (chrony o ntp):** Para mantener el reloj sincronizado.
    ```bash
    emerge net-misc/chrony # O net-misc/ntp
    rc-update add chronyd default # O ntpd
    ```
3.  **Servicios LVM y Crypt:** Necesarios para que el sistema gestione los volúmenes y la encriptación al inicio.
    ```bash
    rc-update add lvm boot
    rc-update add dmcrypt boot
    ```
4.  **(Opcional) Servidor SSH:** Si quieres acceder remotamente.
    ```bash
    emerge net-misc/openssh
    rc-update add sshd default
    # Considera configurar /etc/ssh/sshd_config para mayor seguridad (ej. deshabilitar root login, usar llaves)
    ```
5.  **(Opcional) Herramientas BTRFS:** Ya deberían estar, pero por si acaso.
    ```bash
    emerge sys-fs/btrfs-progs
    ```
6.  **(Opcional) Sudo/Doas:** Para permitir a tu usuario ejecutar comandos como root.
    ```bash
    emerge app-admin/sudo # O app-admin/doas
    # Configura /etc/sudoers (con 'visudo') o /etc/doas.conf para permitir al grupo 'wheel'
    # Ejemplo para sudoers: descomenta '%wheel ALL=(ALL:ALL) ALL'
    ```
7.  **(Opcional) Shell ZSH y Completions:**
    ```bash
    emerge app-shells/zsh app-shells/gentoo-zsh-completion
    # Cambia el shell para root y tu usuario (opcional)
    chsh -s /bin/zsh root
    chsh -s /bin/zsh miusuario
    ```

#### Configuración del Cargador de Arranque (GRUB)

GRUB necesita parches especiales para manejar LUKS2 con Argon2id desde el propio cargador.

1.  **Instalar Dependencias y Parches:**
    *   Asegúrate de que `cryptsetup` se compiló *sin* `static-libs` y *con* `argon2`. Modifica `/etc/portage/package.use/` si es necesario y re-emerge `cryptsetup`.
    ```bash
    echo "sys-fs/cryptsetup argon2 -static-libs" >> /etc/portage/package.use/cryptsetup
    emerge sys-fs/cryptsetup
    ```
    *   Crea directorios para parches y descarga los parches necesarios (los enlaces pueden cambiar, busca "grub luks2 argon2 gentoo patch"):
    ```bash
    mkdir -p /etc/portage/patches/sys-boot/grub:2 # :2 indica slot, ajusta si usas otro
    cd /etc/portage/patches/sys-boot/grub:2
    # ¡VERIFICA ESTOS ENLACES! PUEDEN ESTAR DESACTUALIZADOS.
    curl -O https://gitlab.com/alebeta/gentoo-grub-luks2/-/raw/main/sys-boot/grub/files/5000-grub-2.06-luks2-argon2-v5.patch
    # Puede que necesites más parches dependiendo de la versión de GRUB
    cd ~
    ```
2.  **Instalar GRUB:** Portage debería aplicar los parches automáticamente.
    ```bash
    emerge sys-boot/grub
    ```
3.  **Instalar GRUB en el Disco:**
    *   **Para UEFI:**
        *   Asegúrate de que las variables EFI estén montadas: `mount -t efivarfs efivarfs /sys/firmware/efi/efivars` (normalmente ya está).
        ```bash
        grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo --removable
        # --removable es útil para compatibilidad y USBs. --bootloader-id es el nombre en el menú UEFI.
        ```
    *   **Para BIOS:** (Reemplaza `$DRIVE` con tu disco, ej `/dev/sda`)
        ```bash
        grub-install --target=i386-pc $DRIVE
        ```
4.  **Configurar GRUB (`/etc/default/grub`):** Edita este archivo para añadir parámetros al kernel.
    ```bash
    nano /etc/default/grub
    ```
    Modifica/Añade estas líneas:
    ```ini
    # Habilita el soporte para discos encriptados en GRUB
    GRUB_ENABLE_CRYPTODISK=y

    # Parámetros a pasar al Kernel Linux
    # rd.luks.uuid: UUID de la partición LUKS (sin 'luks-')
    # rd.lvm.lv: Nombre completo del LV raíz
    # cryptdevice: Mapeo de UUID a nombre lógico (como en crypttab)
    # root: Dispositivo raíz final
    # rootfstype: Tipo de sistema de archivos raíz
    # resume=UUID=... : Para hibernación (usa el UUID del swapfile)
    # resume_offset=... : Offset para BTRFS (si lo calculaste)
    # quiet: Menos mensajes de arranque
    # loglevel=3: Nivel de logs del kernel
    GRUB_CMDLINE_LINUX="rd.luks.uuid=${LUKS_UUID} rd.lvm.lv=${VG_NAME}/${LV_NAME} cryptdevice=UUID=${LUKS_UUID}:cryptroot root=/dev/mapper/${VG_NAME}-${LV_NAME} rootfstype=btrfs quiet loglevel=3"
    # Añade resume=UUID=${SWAP_UUID} y resume_offset=${RESUME_OFFSET} si quieres hibernar
    ```
5.  **Generar Configuración de GRUB:**
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
    *   Revisa la salida por errores. Debería detectar tu kernel Linux y añadir los parámetros correctos.

#### Reinicio Inicial

¡El momento de la verdad!

1.  Sal del chroot: `exit`
2.  Desmonta todo en orden inverso (con cuidado):
    ```bash
    cd /
    umount -R /mnt/gentoo/boot # Si montaste /boot/efi por separado, desmonta eso primero
    umount -R /mnt/gentoo
    # Desactiva LVM y LUKS
    vgchange -an ${VG_NAME}
    cryptsetup close cryptroot
    ```
3.  Reinicia: `reboot`
4.  **Importante:** Retira el medio de instalación (USB/ISO). Asegúrate de que el BIOS/UEFI esté configurado para arrancar desde el disco duro interno.

Deberías ver el menú de GRUB, luego te pedirá la contraseña LUKS. Si todo va bien, Gentoo arrancará.

