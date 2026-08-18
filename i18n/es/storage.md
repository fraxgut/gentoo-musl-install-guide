<!--
i18n/es/storage.md
@fraxgut
CC-BY-SA-4.0
Storage layout: partitioning, LUKS2, Btrfs subvolumes and the swapfile
-->

# Arquitectura de almacenamiento

> 🌐 **Idioma:** [English](../en/storage.md) · **Español**

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
        export DISK_LABEL="gentoosys" # Elige un nombre corto y descriptivo
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
        export DISK_LABEL="gentoosys" # Elige un nombre corto y descriptivo
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
    export BTRFS_LABEL="gentoobtrfs"
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
    # Este comando debería dar error si el CoW se aplicó correctamente
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

