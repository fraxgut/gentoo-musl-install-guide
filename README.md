<!--
README.md
@fraxgut
VPL+ACR
Guía de instalación de Gentoo
-->

<a name="readme-top"></a>

<!--
*** Gracias por revisar esta guía de instalación de Gentoo.
*** Si tienes sugerencias para mejorarla, por favor haz un fork y crea un Pull Request
*** o abre un issue con la etiqueta "enhancement".
*** ¡No olvides darle una estrella al proyecto! ¡Gracias!
-->

<!-- ESCUDOS PROYECTO -->
<p align="center">
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo/graphs/contributors">
    <img src="https://img.shields.io/github/contributors/fraxgut/guia-instalacion-gentoo.svg?style=for-the-badge" alt="Contribuidores"/>
  </a>
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo/network/members">
    <img src="https://img.shields.io/github/forks/fraxgut/guia-instalacion-gentoo.svg?style=for-the-badge" alt="Forks"/>
  </a>
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo/stargazers">
    <img src="https://img.shields.io/github/stars/fraxgut/guia-instalacion-gentoo.svg?style=for-the-badge" alt="Stars"/>
  </a>
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo/issues">
    <img src="https://img.shields.io/github/issues/fraxgut/guia-instalacion-gentoo.svg?style=for-the-badge" alt="Issues"/>
  </a>
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo/blob/main/LICENCE">
    <img src="https://img.shields.io/badge/LICENCE-LPC-%2362A98B?style=for-the-badge" alt="Licence"/>
  </a>
  <a href="https://linkedin.com/in/fraxgut">
    <img src="https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&amp;logo=linkedin&amp;colorB=555" alt="LinkedIn"/>
  </a>
</p>


<!-- LOGO PROYECTO -->
<br />
<div align="center">
  <a href="https://github.com/fraxgut/guia-instalacion-gentoo">
    <img src="images/Logo.png" alt="Logo" width="80" height="80">
  </a>

<h3 align="center">Guía Avanzada de Instalación de Gentoo GNU/Linux</h3>

  <p align="center">
    <strong>Variante:</strong> AMD64 Hardened MUSL (BTRFS + LUKS + LVM + LTO + LLVM + Zen)
    <br />
    <em>Una guía detallada para usuarios experimentados que buscan una configuración específica y optimizada.</em>
    <br />
    <br />
    <a href="https://github.com/fraxgut/guia-instalacion-gentoo"><strong>Explorar el repositorio »</strong></a>
    <br />
    <br />
    <a href="https://github.com/fraxgut/guia-instalacion-gentoo/issues">Reportar Errores</a>
    ·
    <a href="https://github.com/fraxgut/guia-instalacion-gentoo/issues">Solicitar Mejoras</a>
  </p>
</div>

<!-- ÍNDICE -->
<details open>
  <summary>Índice</summary>
  <ol>
    <li><a href="#sobre-el-proyecto">Sobre el Proyecto</a>
      <ul>
        <li><a href="#audiencia-objetivo">Audiencia Objetivo</a></li>
        <li><a href="#objetivos-de-esta-configuración">Objetivos de esta Configuración</a></li>
      </ul>
    </li>
    <li><a href="#prerequisitos">Prerequisitos</a>
      <ul>
        <li><a href="#conocimientos-previos">Conocimientos Previos</a></li>
        <li><a href="#descargar-un-sistema-para-la-instalación">Descargar un Sistema para la Instalación</a></li>
        <li><a href="#definir-el-medio-de-instalación">Definir el Medio de Instalación</a></li>
        <li><a href="#grabar-la-imagen-caso-1">Grabar la Imagen (Caso 1)</a></li>
        <li><a href="#máquina-virtual-caso-2">Máquina Virtual (Caso 2)</a></li>
        <li><a href="#conexión-remota-caso-3">Conexión Remota (Caso 3)</a></li>
      </ul>
    </li>
    <li><a href="#instalación">Instalación</a>
      <ul>
        <li><a href="#nota-sobre-los-marcadores-x">Nota sobre los Marcadores [x]</a></li>
        <li><a href="#paso-1-preparación-del-entorno-de-instalación">Paso 1: Preparación del Entorno de Instalación</a></li>
        <li><a href="#paso-2-configuración-del-disco">Paso 2: Configuración del Disco</a>
          <ul>
             <li><a href="#identificación-y-limpieza-del-disco">Identificación y Limpieza del Disco</a></li>
             <li><a href="#particionado-uefi-o-bios">Particionado (UEFI o BIOS)</a></li>
             <li><a href="#encriptación-luks">Encriptación LUKS</a></li>
             <li><a href="#configuración-de-lvm">Configuración de LVM</a></li>
             <li><a href="#formateo-de-particiones">Formateo de Particiones</a></li>
             <li><a href="#configuración-de-btrfs-y-subvolúmenes">Configuración de BTRFS y Subvolúmenes</a></li>
             <li><a href="#montaje-de-sistemas-de-archivos">Montaje de Sistemas de Archivos</a></li>
             <li><a href="#creación-y-activación-del-swapfile">Creación y Activación del Swapfile</a></li>
          </ul>
        </li>
        <li><a href="#paso-3-instalación-del-sistema-base-gentoo">Paso 3: Instalación del Sistema Base Gentoo</a>
            <ul>
              <li><a href="#descarga-y-verificación-del-stage3">Descarga y Verificación del Stage3</a></li>
              <li><a href="#extracción-del-stage3-y-configuración-de-portage">Extracción del Stage3 y Configuración de Portage</a></li>
              <li><a href="#montajes-adicionales-y-entrada-al-chroot">Montajes Adicionales y Entrada al Chroot</a></li>
              <li><a href="#configuración-inicial-dentro-del-chroot">Configuración Inicial dentro del Chroot</a></li>
              <li><a href="#configuración-de-portage-makeconf-y-repositorios">Configuración de Portage (make.conf y Repositorios)</a></li>
              <li><a href="#configuración-de-musl-localización-y-zona-horaria">Configuración de MUSL (Localización y Zona Horaria)</a></li>
            </ul>
        </li>
        <li><a href="#paso-4-compilación-del-kernel">Paso 4: Compilación del Kernel</a>
            <ul>
              <li><a href="#instalación-de-fuentes-y-firmware">Instalación de Fuentes y Firmware</a></li>
              <li><a href="#configuración-del-kernel-zen">Configuración del Kernel (Zen)</a></li>
              <li><a href="#compilación-e-instalación-del-kernel-con-dracut">Compilación e Instalación del Kernel con Dracut</a></li>
            </ul>
        </li>
        <li><a href="#paso-5-configuración-final-del-sistema">Paso 5: Configuración Final del Sistema</a>
            <ul>
              <li><a href="#configuración-de-fstab-y-crypttab">Configuración de fstab y crypttab</a></li>
              <li><a href="#configuración-de-red-hostname-y-usuarios">Configuración de Red, Hostname y Usuarios</a></li>
              <li><a href="#servicios-esenciales-y-herramientas">Servicios Esenciales y Herramientas</a></li>
              <li><a href="#configuración-del-cargador-de-arranque-grub">Configuración del Cargador de Arranque (GRUB)</a></li>
              <li><a href="#reinicio-inicial">Reinicio Inicial</a></li>
            </ul>
        </li>
        <li><a href="#paso-6-configuración-post-instalación-llvm-y-lto">Paso 6: Configuración Post-Instalación (LLVM y LTO)</a>
            <ul>
              <li><a href="#configuración-de-llvmclang-como-compilador-del-sistema">Configuración de LLVM/Clang como Compilador del Sistema</a></li>
              <li><a href="#compilación-del-kernel-con-llvmclang">Compilación del Kernel con LLVM/Clang</a></li>
              <li><a href="#activación-de-optimizaciones-lto-gentoolto">Activación de Optimizaciones LTO (GentooLTO)</a></li>
            </ul>
        </li>
      </ul>
    </li>
    <li><a href="#solución-de-problemas-comunes">Solución de Problemas Comunes</a></li>
    <li><a href="#hoja-de-ruta">Hoja de ruta</a></li>
    <li><a href="#contribuir">Contribuir</a></li>
    <li><a href="#licencia">Licencia</a></li>
    <li><a href="#contacto">Contacto</a></li>
    <li><a href="#reconocimientos">Reconocimientos</a></li>
  </ol>
</details>

<!-- SOBRE EL PROYECTO -->
## Sobre el proyecto

Esta guía proporciona un camino detallado para instalar una variante específica y avanzada de Gentoo GNU/Linux. No es una guía introductoria general, sino un tutorial paso a paso enfocado en la combinación de tecnologías mencionada.

**La configuración resultante busca:**

*   **Seguridad:** Mediante el perfil "Hardened" y la encriptación de disco completo (LUKS). (La configuración de SELinux está planeada).
*   **Flexibilidad y Modernidad:** Usando BTRFS para instantáneas y gestión de volúmenes flexible, combinado con LVM.
*   **Alternativa a glibc:** Empleando MUSL como librería C estándar, conocida por ser más ligera y simple.
*   **Rendimiento y Optimización:** Utilizando el kernel Zen, el compilador LLVM/Clang y optimizaciones LTO (Link-Time Optimization) para intentar obtener el máximo rendimiento del hardware.

### Audiencia Objetivo

Esta guía está dirigida a **usuarios experimentados de Linux y Gentoo**. Se asume familiaridad con la línea de comandos, conceptos de particionado, sistemas de archivos, compilación de software (especialmente el kernel) y la filosofía general de Gentoo (Portage, USE flags, etc.). **No se recomienda para principiantes en Gentoo.**

### Objetivos de esta Configuración

*   **Sistema Base Robusto:** Establecer una base Gentoo Hardened con MUSL.
*   **Almacenamiento Avanzado:** Implementar LUKS sobre LVM sobre BTRFS.
*   **Toolchain Moderno:** Usar LLVM/Clang como compilador principal.
*   **Optimización Agresiva:** Aplicar LTO a nivel de sistema.
*   **Kernel Optimizado:** Utilizar el kernel Zen.

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- PREREQUISITOS -->
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

### Paso 2: Configuración del Disco

Esta sección prepara el disco con encriptación LUKS, LVM y BTRFS.

#### Identificación y Limpieza del Disco

1.  Identifica el disco de destino (ej: `/dev/sda`, `/dev/nvme0n1`). ¡Ten cuidado!
    ```bash
    lsblk
    fdisk -l
    ```
2.  Establece la variable (¡**REEMPLAZA** `sdx` o `nvme0n1pX` con tu disco!):
    ```bash
    export DRIVE=/dev/sdx
    ```
3.  **¡ADVERTENCIA: BORRADO IRREVERSIBLE DEL DISCO!** Los siguientes comandos destruyen todos los datos en `$DRIVE`.
    *   (Opcional pero recomendado) Sobrescribir con datos aleatorios para ocultar patrones LUKS:
        ```bash
        cryptsetup open --type plain $DRIVE container --key-file /dev/urandom
        dd if=/dev/urandom of=/dev/mapper/container status=progress bs=1M
        cryptsetup close container
        ```
    *   Limpiar tablas de particiones existentes:
        ```bash
        sgdisk --zap-all $DRIVE
        ```

#### Particionado (UEFI o BIOS)

1.  Detecta el modo de arranque:
    ```bash
    [ -d /sys/firmware/efi ] && echo "UEFI detectado" || echo "BIOS (Legacy) detectado"
    ```
2.  Crea las particiones (Elige **UNO** de los siguientes bloques):
    *   **Para Sistemas UEFI:**
        ```bash
        # /dev/sdx1: EFI System Partition (ESP), ~1GiB, FAT32
        # /dev/sdx2: Partición para LUKS (resto del disco), Linux Filesystem
        export DISK_LABEL="GentooSys" # Elige un nombre corto y descriptivo
        sgdisk --clear \
               --new=1:0:+1GiB --typecode=1:ef00 --change-name=1:EFI \
               --new=2:0:0   --typecode=2:8300 --change-name=2:${DISK_LABEL} \
               $DRIVE
        export LUKS_PART_LABEL=${DISK_LABEL}
        export EFI_PART_LABEL=EFI
        ```
    *   **Para Sistemas BIOS (Legacy):**
        ```bash
        # /dev/sdx1: BIOS Boot Partition (para GRUB), ~1MiB, tipo ef02
        # /dev/sdx2: Partición /boot separada, ~1GiB, ext2/ext4
        # /dev/sdx3: Partición para LUKS (resto del disco), Linux Filesystem
        export DISK_LABEL="GentooSys" # Elige un nombre corto y descriptivo
        sgdisk --clear \
               --new=1:0:+1MiB  --typecode=1:ef02 --change-name=1:GRUB \
               --new=2:0:+1GiB  --typecode=2:8300 --change-name=2:BOOT \
               --new=3:0:0    --typecode=3:8300 --change-name=3:${DISK_LABEL} \
               $DRIVE
        export LUKS_PART_LABEL=${DISK_LABEL}
        export BOOT_PART_LABEL=BOOT
        ```

#### Encriptación LUKS

1.  Crea el contenedor LUKS en la partición designada. Elige una **contraseña muy segura**.
    ```bash
    cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha512 \
               --iter-time 5000 --key-size 512 --pbkdf argon2id \
               --use-random --verify-passphrase \
               luksFormat /dev/disk/by-partlabel/${LUKS_PART_LABEL}
    ```
2.  Abre el contenedor LUKS (se te pedirá la contraseña):
    ```bash
    cryptsetup open /dev/disk/by-partlabel/${LUKS_PART_LABEL} cryptroot
    # Ahora tendrás /dev/mapper/cryptroot
    ```
    *Marcador:* `[x]` Si reinicias, necesitas reabrir el contenedor LUKS.

#### Configuración de LVM

Se crea un Grupo de Volúmenes (VG) y un Volumen Lógico (LV) dentro del contenedor LUKS abierto.

1.  Crea el Volumen Físico (PV) sobre el dispositivo mapeado:
    ```bash
    pvcreate /dev/mapper/cryptroot
    ```
2.  Crea el Grupo de Volúmenes (VG). Elige un nombre (ej. `vg_gentoo`):
    ```bash
    export VG_NAME="vg_gentoo"
    vgcreate ${VG_NAME} /dev/mapper/cryptroot
    ```
3.  Crea el Volumen Lógico (LV) que ocupará todo el espacio disponible en el VG. Elige un nombre (ej. `lv_root`):
    ```bash
    export LV_NAME="lv_root"
    lvcreate -l +100%FREE ${VG_NAME} --name ${LV_NAME}
    # Ahora tendrás /dev/mapper/vg_gentoo-lv_root (o similar)
    ```

#### Formateo de Particiones

1.  Formatea las particiones según el esquema (UEFI o BIOS):
    *   **Para UEFI:** Formatea la partición EFI como FAT32.
        ```bash
        mkfs.vfat -F 32 -n EFI /dev/disk/by-partlabel/${EFI_PART_LABEL}
        ```
    *   **Para BIOS:** Formatea la partición `/boot` (se recomienda `ext4` por journaling, pero `ext2` es más simple si no necesitas esa robustez aquí).
        ```bash
        mkfs.ext4 -L BOOT /dev/disk/by-partlabel/${BOOT_PART_LABEL}
        # O: mkfs.ext2 -L BOOT /dev/disk/by-partlabel/${BOOT_PART_LABEL}
        ```
2.  Formatea el Volumen Lógico LVM como BTRFS:
    ```bash
    export BTRFS_LABEL="GentooBTRFS"
    mkfs.btrfs --force --label ${BTRFS_LABEL} /dev/${VG_NAME}/${LV_NAME}
    ```

#### Configuración de BTRFS y Subvolúmenes

Montamos temporalmente BTRFS para crear la estructura de subvolúmenes recomendada.

1.  Monta la raíz BTRFS temporalmente:
    ```bash
    mkdir -p /mnt/gentoo
    mount -t btrfs LABEL=${BTRFS_LABEL} /mnt/gentoo
    ```
2.  Crea los subvolúmenes principales:
    *   `@`: Será la raíz real (`/`) del sistema instalado.
    *   `@home`: Para los directorios de usuario (`/home`).
    *   `@snapshots`: Para instantáneas BTRFS (útil para backups/rollback).
    *   `@<otros>`: Puedes crear más para `/var/log`, `/var/tmp`, etc., si lo deseas. Esta guía crea algunos básicos.
    ```bash
    btrfs subvolume create /mnt/gentoo/@
    btrfs subvolume create /mnt/gentoo/@home
    btrfs subvolume create /mnt/gentoo/@snapshots
    btrfs subvolume create /mnt/gentoo/@vartmp # Para /var/tmp
    btrfs subvolume create /mnt/gentoo/@varlog # Para /var/log (opcional)
    btrfs subvolume create /mnt/gentoo/@swap   # Contendrá el swapfile
    # Considera otros como @portage, @srv, @opt si separas mucho
    ```
3.  Desmonta la raíz BTRFS temporal:
    ```bash
    umount -R /mnt/gentoo
    ```

#### Montaje de Sistemas de Archivos

Ahora montamos los subvolúmenes y particiones en la estructura final bajo `/mnt/gentoo`.

1.  Define opciones de montaje comunes para BTRFS (ajusta `compress` o `ssd` según tu hardware):
    ```bash
    export MOUNT_OPTS_BTRFS="defaults,compress-force=zstd:3,ssd,noatime,space_cache=v2,subvol="
    ```
    *Marcador:* `[x]` Redefinir `MOUNT_OPTS_BTRFS` si la sesión se reinicia.
2.  Monta el subvolumen raíz (`@`):
    ```bash
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@ LABEL=${BTRFS_LABEL} /mnt/gentoo
    ```
    *Marcador:* `[x]` Punto de montaje principal.
3.  Crea los puntos de montaje *dentro* de `/mnt/gentoo` para los otros subvolúmenes y particiones:
    ```bash
    mkdir -p /mnt/gentoo/{boot,home,.snapshots,var/tmp,var/log,var/swap}
    # Crea otros directorios si creaste más subvolúmenes (ej: /mnt/gentoo/srv)
    ```
4.  Monta los demás subvolúmenes BTRFS:
    ```bash
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@home LABEL=${BTRFS_LABEL} /mnt/gentoo/home
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@snapshots LABEL=${BTRFS_LABEL} /mnt/gentoo/.snapshots
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@vartmp LABEL=${BTRFS_LABEL} /mnt/gentoo/var/tmp
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@varlog LABEL=${BTRFS_LABEL} /mnt/gentoo/var/log # Si lo creaste
    mount -t btrfs -o ${MOUNT_OPTS_BTRFS}@swap LABEL=${BTRFS_LABEL} /mnt/gentoo/var/swap
    # Monta otros si los creaste
    ```
    *Marcador:* `[x]` Montajes BTRFS.
5.  Monta la partición de arranque (EFI o /boot):
    *   **Para UEFI:**
        ```bash
        # Opciones para FAT32 (EFI)
        export MOUNT_OPTS_EFI="rw,nosuid,nodev,noexec,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro"
        mount -o ${MOUNT_OPTS_EFI} /dev/disk/by-partlabel/${EFI_PART_LABEL} /mnt/gentoo/boot
        # Necesitamos el directorio efi dentro de boot
        mkdir -p /mnt/gentoo/boot/efi
        ```
    *   **Para BIOS:**
        ```bash
        # Opciones para ext4/ext2 (/boot)
        export MOUNT_OPTS_BOOT="rw,nosuid,nodev,relatime"
        mount -o ${MOUNT_OPTS_BOOT} /dev/disk/by-partlabel/${BOOT_PART_LABEL} /mnt/gentoo/boot
        ```
    *Marcador:* `[x]` Montaje de /boot.
6.  Ajusta permisos para directorios temporales:
    ```bash
    chmod 1777 /mnt/gentoo/var/tmp
    # Si no usas subvolumen para /tmp, haz: mkdir /mnt/gentoo/tmp && chmod 1777 /mnt/gentoo/tmp
    ```

#### Creación y Activación del Swapfile

Usaremos un archivo dentro del subvolumen `@swap` como espacio de intercambio.

| Tamaño RAM | Tamaño del "swapfile" (Sin Hibernar) | Tamaño del "swapfile" (Con Hibernar) |
|------------|--------------------------------------|--------------------------------------|
| 256MB      | 256MB                                | 512MB                                |
| 512MB      | 512MB                                | 1GB                                  |
| 1GB        | 1GB                                  | 2GB                                  |
| 2GB        | 1GB                                  | 3GB                                  |
| 3GB        | 2GB                                  | 5GB                                  |
| 4GB        | 2GB                                  | 6GB                                  |
| 6GB        | 2GB                                  | 8GB                                  |
| 8GB        | 3GB                                  | 11GB                                 |
| 12GB       | 3GB                                  | 15GB                                 |
| 16GB       | 4GB                                  | 20GB                                 |
| 24GB       | 5GB                                  | 29GB                                 |
| 32GB       | 6GB                                  | 38GB                                 |
| 64GB       | 8GB                                  | 72GB                                 |
| 128GB      | 11GB                                 | 139GB                                |


1.  **Define el Tamaño:** Elige el tamaño según tu RAM y si usarás hibernación. La tabla proporcionada es una buena guía. Ejemplo para 8GB RAM sin hibernar (3GB):
    ```bash
    export SWAP_SIZE_GB=3
    ```
2.  Crea el archivo swap vacío y deshabilita CoW (Copy-on-Write) y compresión para él:
    ```bash
    truncate -s 0 /mnt/gentoo/var/swap/swapfile
    chattr +C /mnt/gentoo/var/swap/swapfile # Deshabilita CoW (IMPORTANTE)
    # Verifica que no tenga compresión (debería heredar 'none' si el subvol lo tiene, pero por si acaso)
    btrfs property set /mnt/gentoo/var/swap/swapfile compression none
    ```
3.  Asigna el espacio (usa `dd`). Asegúrate de que `bs*count` iguale el tamaño deseado:
    ```bash
    dd if=/dev/zero of=/mnt/gentoo/var/swap/swapfile bs=1G count=${SWAP_SIZE_GB} status=progress
    # O usa bs=1M y ajusta 'count' si necesitas más granularidad
    ```
4.  Establece permisos correctos:
    ```bash
    chmod 600 /mnt/gentoo/var/swap/swapfile
    ```
5.  Formatea el archivo como swap:
    ```bash
    mkswap -L swap /mnt/gentoo/var/swap/swapfile
    ```
6.  Activa el swap:
    ```bash
    swapon /mnt/gentoo/var/swap/swapfile
    # Verifica con: swapon --show o free -h
    ```
    *Marcador:* `[x]` Swap activado.
7.  Navega al punto de montaje raíz:
    ```bash
    cd /mnt/gentoo
    ```
    *Marcador:* `[x]` Directorio actual.

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
    COMMON_FLAGS="-march=native -O2 -pipe"
    CFLAGS="${COMMON_FLAGS}"
    CXXFLAGS="${COMMON_FLAGS}"
    FCFLAGS="${COMMON_FLAGS}" # Fortran
    FFLAGS="${COMMON_FLAGS}"  # Fortran

    # Arquitectura y Host (ya debería estar correcto por el stage3)
    CHOST="x86_64-gentoo-linux-musl"

    # --- Opciones de Paralelización ---
    # NPROC = Número de hilos de CPU. 'nproc' lo detecta.
    NPROC=$(nproc)
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

#### Configuración de `fstab` y `crypttab`

Estos archivos le dicen al sistema cómo montar los discos al arrancar.

1.  **Obtener UUIDs:** Necesitamos los identificadores únicos de las particiones y dispositivos.
    ```bash
    # UUID de la partición LUKS
    export LUKS_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${LUKS_PART_LABEL})
    # UUID del volumen lógico BTRFS (raíz)
    export ROOT_UUID=$(blkid -s UUID -o value /dev/${VG_NAME}/${LV_NAME})
    # UUID del archivo swap
    export SWAP_UUID=$(blkid -s UUID -o value /mnt/gentoo/var/swap/swapfile) # Asegúrate que esté montado

    # UUID de la partición de arranque
    if [ -d /sys/firmware/efi ]; then # UEFI
      export BOOT_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${EFI_PART_LABEL})
      export BOOT_FSTYPE="vfat"
      export BOOT_OPTIONS="${MOUNT_OPTS_EFI}" # Usa las opciones definidas antes
    else # BIOS
      export BOOT_UUID=$(blkid -s UUID -o value /dev/disk/by-partlabel/${BOOT_PART_LABEL})
      export BOOT_FSTYPE="ext4" # O ext2 si usaste eso
      export BOOT_OPTIONS="${MOUNT_OPTS_BOOT}" # Usa las opciones definidas antes
    fi

    # Offset para hibernación (si planeas usarla, requiere cálculo específico para BTRFS)
    # export RESUME_OFFSET=$(btrfs inspect-internal map-swapfile -r /var/swap/swapfile) # Requiere que @swap esté montado en /var/swap
    ```
    *Marcador:* `[x]` Asegúrate de tener los UUIDs correctos.
2.  **Crear `/etc/crypttab`:** Le dice a `systemd-cryptsetup` (o equivalente OpenRC) cómo abrir el LUKS.
    ```bash
    # Nombre lógico | Dispositivo UUID | Password File | Opciones
    echo "cryptroot UUID=${LUKS_UUID} none luks,discard" > /etc/crypttab
    # 'none' significa que pedirá la pass al inicio. 'discard' es para SSDs.
    ```
3.  **Crear `/etc/fstab`:** Define los puntos de montaje. **¡Revisa cuidadosamente!**
    ```bash
    # Define opciones BTRFS base para fstab
    export FSTAB_BTRFS_OPTS="defaults,compress-force=zstd:3,ssd,noatime,space_cache=v2"

    cat <<EOF > /etc/fstab
	# <filesystem>                             <mountpoint>  <type>  <options>                               <dump> <pass>
	# Raíz (subvolumen @)
	UUID=${ROOT_UUID}                          /             btrfs   ${FSTAB_BTRFS_OPTS},subvol=@             0      1
	
	# Partición de Arranque (/boot)
	UUID=${BOOT_UUID}                          /boot         ${BOOT_FSTYPE} ${BOOT_OPTIONS}                     0      2
	
	# Subvolumen Home (@home)
	UUID=${ROOT_UUID}                          /home         btrfs   ${FSTAB_BTRFS_OPTS},subvol=@home         0      2
	
	# Subvolumen Snapshots (@snapshots)
	UUID=${ROOT_UUID}                          /.snapshots   btrfs   ${FSTAB_BTRFS_OPTS},subvol=@snapshots     0      2
	
	# Subvolumen VarTmp (@vartmp)
	UUID=${ROOT_UUID}                          /var/tmp      btrfs   ${FSTAB_BTRFS_OPTS},subvol=@vartmp       0      2
	
	# Subvolumen VarLog (@varlog) - si lo creaste
	UUID=${ROOT_UUID}                          /var/log      btrfs   ${FSTAB_BTRFS_OPTS},subvol=@varlog       0      2
	
	# Subvolumen Swap (@swap) - Montado para que el swapfile sea accesible
	UUID=${ROOT_UUID}                          /var/swap     btrfs   ${FSTAB_BTRFS_OPTS},subvol=@swap,nodatacow 0      2
	
	# Swapfile (referenciado por UUID del archivo)
	UUID=${SWAP_UUID}                          none          swap    sw                                        0      0
	
	# tmpfs para /tmp (si no usas subvolumen)
	# tmpfs                                    /tmp          tmpfs   defaults,nosuid,nodev                   0      0
	EOF
	```
    *   **Nota:** El montaje de `@swap` es principalmente para que el sistema encuentre el `swapfile` por su UUID. `nodatacow` es importante si no lo pusiste con `chattr +C`.
    *   Revisa el archivo: `cat /etc/fstab`

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

### Paso 6: Configuración Post-Instalación (LLVM y LTO)

Estos pasos son **opcionales** y muy avanzados. Convierten el sistema para usar Clang/LLVM como compilador principal y aplican optimizaciones LTO. **Esto puede introducir inestabilidad y aumentar significativamente los tiempos de compilación.** Procede con precaución.

#### Configuración de LLVM/Clang como Compilador del Sistema

1.  **Instalar Paquetes LLVM/Clang:**
    *   Añade USE flags necesarias para libcxx/libunwind.
    ```bash
    echo "sys-libs/llvm-libunwind static-libs" >> /etc/portage/package.use/llvm
    echo "sys-libs/libcxx static-libs" >> /etc/portage/package.use/llvm
    echo "sys-libs/libcxxabi static-libs" >> /etc/portage/package.use/llvm
    emerge sys-devel/clang sys-devel/llvm sys-libs/compiler-rt sys-libs/llvm-libunwind sys-devel/lld sys-libs/libcxx sys-libs/libcxxabi
    ```
2.  **Añadir Overlays Específicos:** `toolchain-clang` y `clang-musl-overlay`.
    ```bash
    eselect repository add toolchain-clang git https://github.com/2b57/toolchain-clang.git
    # Crea el archivo de configuración para clang-musl-overlay
    cat <<EOF > /etc/portage/repos.conf/clang-musl.conf
    [clang-musl]
    location = /var/db/repos/clang-musl
    sync-type = git
    sync-uri = https://github.com/clang-musl-overlay/clang-musl-overlay.git
    sync-depth = 1
    auto-sync = yes
    EOF
    emerge --sync # Sincroniza los nuevos repos
    ```
3.  **Seleccionar Perfil Clang/MUSL:** Cambia al perfil que usa Clang.
    ```bash
    eselect profile list
    # Busca y selecciona el perfil .../clang/musl/hardened (o similar)
    # eselect profile set --force <NUMERO_PERFIL_CLANG>
    ```
4.  **Configurar Entorno para Clang:**
    *   Crea un archivo de entorno para el kernel:
    ```bash
    mkdir -p /etc/portage/env
    echo "LLVM=1 LLVM_IAS=1" > /etc/portage/env/kernel-clang
    # Asocia este entorno a los paquetes del kernel
    mkdir -p /etc/portage/package.env
    echo "sys-kernel/* kernel-clang" >> /etc/portage/package.env/kernel
    ```
    *   Añade workarounds si son necesarios (ejemplo para LTO):
    ```bash
    mkdir -p /etc/portage/package.cflags
    # echo "sys-devel/llvm *FLAGS-="-fipa-pta"" >> /etc/portage/package.cflags/ltoworkarounds.conf
    ```
5.  **Reconstruir Toolchain con Clang:**
    *   Limpia paquetes GCC si el perfil lo requiere (revisa la salida de `emerge`): `emerge -c gcc` (¡con cuidado!)
    *   Reinstala los componentes clave de LLVM/Clang usando Clang.
    ```bash
    emerge --keep-going sys-devel/clang sys-devel/llvm sys-libs/compiler-rt sys-libs/llvm-libunwind sys-devel/lld
    ```
6.  **Actualizar `make.conf` para Usar Clang/LLD:**
    ```bash
    nano /etc/portage/make.conf
    ```
    Ajusta/Añade las variables de compilación:
    ```ini
    # ... (otras variables)
    CC="clang"
    CXX="clang++"
    AR="llvm-ar"
    NM="llvm-nm"
    RANLIB="llvm-ranlib"
    # CFLAGS y CXXFLAGS ya definidos (-march=native -O2 -pipe)

    # Configuración de LDFLAGS para usar lld y librerías de Clang
    LDFLAGS="${LDFLAGS} -rtlib=compiler-rt -unwindlib=libunwind -fuse-ld=lld -Wl,--as-needed"

    # ... (resto de make.conf)
    ```
7.  **Actualizar Entorno y Enlaces:**
    ```bash
    env-update && source /etc/profile
    # Configura llvm-conf para crear wrappers (reemplaza X con tu versión de LLVM)
    # emerge sys-devel/llvm-conf # Si no está instalado
    # llvm-conf --enable-shared --enable-assertions --enable-optimized --enable-targets=host --link-shared --bindir=/usr/bin --libdir=/usr/lib64 --includedir=/usr/include --prefix=/usr --sysconfdir=/etc --datadir=/usr/share --infodir=/usr/share/info --mandir=/usr/share/man X
    # O usa los wrappers de eselect-clang si están disponibles
    ```
8.  **Reconstruir el Mundo con Clang:** ¡Esto llevará **MUCHO** tiempo!
    ```bash
    emerge -e @world
    # -e: rebuild everything
    ```

#### Compilación del Kernel con LLVM/Clang

Una vez que el sistema base usa Clang:

```bash
cd /usr/src/linux
# Limpia compilaciones anteriores
make clean
# Configura si es necesario (make menuconfig)
# Compila usando Clang (las variables de entorno deberían funcionar si usaste package.env)
make -j$(nproc) LLVM=1 LLVM_IAS=1
make modules_install LLVM=1 LLVM_IAS=1
make install LLVM=1 LLVM_IAS=1
# Regenera initramfs
KERNEL_VERSION=$(basename /boot/vmlinuz-*) # Re-detecta por si acaso
dracut --force --kver ${KERNEL_VERSION}
# Actualiza GRUB
grub-mkconfig -o /boot/grub/grub.cfg

```
#### Activación de Optimizaciones LTO (GentooLTO)

Esto aplica LTO (Link-Time Optimization) a la mayoría de los paquetes. **Aumenta aún más los tiempos de compilación y el uso de RAM, y el riesgo de problemas.**

1.  **Añadir Overlays LTO:**
    ```bash
    eselect repository enable mv # Dependencia
    eselect repository enable lto-overlay
    emerge --sync
    ```
2.  **Instalar Herramientas LTO:**
    *   Puede requerir aceptar keywords inestables.
    ```bash
    echo "sys-config/ltoize ~amd64" >> /etc/portage/package.accept_keywords/lto
    echo "app-portage/lto-rebuild ~amd64" >> /etc/portage/package.accept_keywords/lto
    emerge sys-config/ltoize app-portage/lto-rebuild
    ```
3.  **Configurar `make.conf` para LTO:**
    *   `ltoize` crea archivos de configuración en `/etc/portage/make.conf/`. El principal es `lto.conf`.
    *   Edita `/etc/portage/make.conf` y añade `source /etc/portage/make.conf/lto.conf` cerca del principio.
    *   Ajusta las `CFLAGS`, `CXXFLAGS`, `LDFLAGS` según las recomendaciones de `ltoize` (normalmente añade flags como `-flto=auto`, `-fno-plt`). **Lee la documentación de GentooLTO.**
    ```bash
    nano /etc/portage/make.conf
    # Añadir al principio:
    # source /etc/portage/make.conf/lto.conf

    # Modificar CFLAGS/CXXFLAGS/LDFLAGS según ltoize:
    # CFLAGS="${COMMON_FLAGS} -flto=auto" # Ejemplo
    # CXXFLAGS="${COMMON_FLAGS} -flto=auto" # Ejemplo
    # LDFLAGS="${LDFLAGS} -flto=auto" # Ejemplo
    ```
4.  **Ejecutar `ltoize`:** Analiza los paquetes instalados y genera configuraciones de USE flags y máscaras para LTO.
    ```bash
    ltoize # Revisa la salida y los archivos generados en /etc/portage/
    ```
5.  **Reconstruir el Mundo con LTO:** ¡Esto será **EXTREMADAMENTE LARGO** y consumirá mucha RAM!
    ```bash
    lto-rebuild -r # Reconstruye paquetes esenciales primero
    emerge -e --keep-going @world # Reconstruye todo lo demás
    ```

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- SOLUCIÓN DE PROBLEMAS -->
## Solución de Problemas Comunes

*   **No arranca GRUB:**
    *   ¿Instalaste GRUB en el disco correcto (`grub-install /dev/sdx`)?
    *   ¿Está el BIOS/UEFI configurado para arrancar desde ese disco?
    *   ¿Usaste el `target` correcto (`i386-pc` para BIOS, `x86_64-efi` para UEFI)?
    *   ¿Está la partición `/boot` (o EFI) formateada correctamente?
*   **GRUB arranca pero no pide contraseña LUKS / Kernel Panic:**
    *   **Problema más común:** El `initramfs` no contiene los módulos necesarios (`crypt`, `lvm`, `btrfs`, drivers de disco/teclado).
        *   Verifica `/etc/dracut.conf`.
        *   Regenera el initramfs: `dracut --force --kver <KERNEL_VERSION>`
        *   Verifica el contenido: `lsinitrd /boot/initramfs-<VERSION>.img | grep -E 'crypt|lvm|btrfs'`
    *   **Parámetros del kernel incorrectos en `/etc/default/grub`:**
        *   Verifica `rd.luks.uuid`, `rd.lvm.lv`, `cryptdevice=UUID=...:cryptroot`, `root=/dev/mapper/...`. Asegúrate de que los UUIDs y nombres sean exactos.
        *   No olvides ejecutar `grub-mkconfig -o /boot/grub/grub.cfg` después de cambiar `/etc/default/grub`.
    *   **Configuración del Kernel:** ¿Faltan drivers esenciales (SATA/NVMe, BTRFS, DM/Crypt) compilados *dentro* del kernel o como módulos *incluidos* en el initramfs?
*   **El sistema arranca pero no monta `/home` u otros subvolúmenes:**
    *   Verifica los UUIDs y las opciones `subvol=` en `/etc/fstab`.
    *   Asegúrate de que los puntos de montaje (`/home`, `/.snapshots`) existen en el subvolumen raíz (`@`).
*   **Errores de compilación (`emerge` falla):**
    *   Lee el mensaje de error detenidamente.
    *   ¿Faltan dependencias? (`emerge -uvDN @world` puede ayudar).
    *   ¿USE flags conflictivas? Revisa `/etc/portage/make.conf` y `/etc/portage/package.use/`.
    *   ¿Problemas con overlays? Asegúrate de que estén sincronizados (`emerge --sync`).
    *   ¿Poca RAM/Swap durante la compilación (especialmente con LTO/Clang)? Intenta reducir `-j` en `MAKEOPTS`.
    *   Busca el error específico en los foros de Gentoo o en el bug tracker.
*   **No hay conexión de red después de reiniciar:**
    *   ¿Está el servicio `NetworkManager` (o el que uses) habilitado? (`rc-update show`)
    *   ¿Están los drivers de red correctos compilados en el kernel (o como módulos)? (`lspci -k`)
    *   Usa `nmtui` para configurar la conexión.

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>



<!-- HOJA DE RUTA -->
## Hoja de Ruta

- [X] Gentoo GNU Linux AMD64
    - [X] BTRFS
    - [X] LVM
    - [X] LUKS
    - [X] Musl
    - [X] LLVM/Clang
    - [X] Zen kernel
    - [X] LTO
- [ ] SELinux (Configuración e integración)
- [ ] Mejoras en la sección de configuración del Kernel (Ej. `.config` de ejemplo)
- [ ] Guía de configuración de hibernación con BTRFS+LUKS+Swapfile

Consulta [Issues](https://github.com/fraxgut/guia-instalacion-gentoo/issues) para ver la lista completa de funciones propuestas y problemas conocidos.

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- CONTRIBUIR -->
## Contribuir

Las contribuciones hacen de la comunidad de código abierto un lugar increíble para aprender, inspirar y crear. Cualquier contribución que realices será **muy apreciada**.

Si tienes sugerencias para mejorar esto, por favor haz un fork del repositorio y crea un pull request. También puedes simplemente abrir un issue con la etiqueta "enhancement".
¡No olvides darle una estrella al proyecto! ¡Gracias de nuevo!

1.  Haz un Fork del Proyecto
2.  Crea tu Feature Branch (`git checkout -b feature/AmazingFeature`)
3.  Haz Commit de tus Cambios (`git commit -m 'Add some AmazingFeature'`)
4.  Haz Push a la Branch (`git push origin feature/AmazingFeature`)
5.  Abre un Pull Request

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- LICENCIA -->
## Licencia

Distribuido bajo la licencia VPL+ACR. Consulta `LICENCE` para más información.

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- CONTACTO -->
## Contacto

Franco Gutiérrez - [@fraxgut](https://twitter.com/fraxgut) - contacto@fraxgut.net

Enlace del Proyecto: [https://github.com/fraxgut/guia-instalacion-gentoo](https://github.com/fraxgut/guia-instalacion-gentoo)

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>

<!-- RECONOCIMIENTOS -->
## Reconocimientos

Recursos y proyectos útiles que inspiraron o se usaron como referencia:

*   [Manual Oficial de Gentoo AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64)
*   [Wiki de Gentoo: Proyecto MUSL](https://wiki.gentoo.org/wiki/Project:Musl)
*   [Wiki de Gentoo: LVM](https://wiki.gentoo.org/wiki/LVM)
*   [Wiki de Gentoo: BTRFS](https://wiki.gentoo.org/wiki/Btrfs)
*   [Wiki de Gentoo: LUKS](https://wiki.gentoo.org/wiki/Dm-crypt/Full_disk_encryption)
*   [Wiki de Gentoo: Dracut](https://wiki.gentoo.org/wiki/Dracut)
*   [Proyecto GURU](https://wiki.gentoo.org/wiki/Project:GURU)
*   [Proyecto GentooLTO](https://github.com/InBetweenNames/gentooLTO)
*   [Overlay toolchain-clang](https://github.com/2b57/toolchain-clang)
*   [Overlay clang-musl-overlay](https://github.com/clang-musl-overlay/clang-musl-overlay)
*   [Plantilla Best-README-Template](https://github.com/othneildrew/Best-README-Template)

<p align="right">(<a href="#readme-top">ir al inicio</a>)</p>
