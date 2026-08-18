<!--
i18n/es/toolchain.md
@fraxgut
CC-BY-SA-4.0
Toolchain: the LLVM/Clang profile, hardening and link-time optimisation
-->

# Cadena de herramientas LLVM y LTO

> 🌐 **Idioma:** [English](../en/toolchain.md) · **Español**

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
