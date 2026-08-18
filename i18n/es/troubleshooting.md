<!--
i18n/es/troubleshooting.md
@fraxgut
CC-BY-SA-4.0
Recovery procedures for boot, storage and build failures
-->

# Solución de problemas

> 🌐 **Idioma:** [English](../en/troubleshooting.md) · **Español**


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
