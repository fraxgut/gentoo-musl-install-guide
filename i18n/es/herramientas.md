<!--
i18n/es/toolchain.md
@fraxgut
CC-BY-SA-4.0
Toolchain: the LLVM/Clang profile, hardening and link-time optimisation
-->

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/instrumenta.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **Español** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/toolchain.md)**

# Cadena de herramientas LLVM y LTO

</div>

---

Este documento explica cómo la instalación combina musl, el endurecimiento del
perfil `hardened` y la cadena de herramientas LLVM/Clang, y cómo se activa la
optimización en tiempo de enlace.

## Índice

1. [El problema de los perfiles](#el-problema-de-los-perfiles)
2. [Qué aporta cada perfil](#qué-aporta-cada-perfil)
3. [El perfil apilado](#el-perfil-apilado)
4. [Fricción conocida](#fricción-conocida)
5. [Configuración de make.conf](#configuración-de-makeconf)
6. [Optimización en tiempo de enlace](#optimización-en-tiempo-de-enlace)
7. [Reconstrucción del sistema](#reconstrucción-del-sistema)
8. [El núcleo con Clang](#el-núcleo-con-clang)

## El problema de los perfiles

Gentoo publica perfiles separados para musl con endurecimiento y para musl con
LLVM:

```
default/linux/amd64/23.0/musl/hardened   (exp)   →  GCC
default/linux/amd64/23.0/musl/llvm       (exp)   →  Clang, LLD, libc++
```

No existe un perfil oficial que los combine, ni un stage3
`musl-llvm-hardened`. La biblioteca C y la cadena de herramientas quedan
fijadas al desempaquetar el stage3. La elección se toma antes de instalar, y
cambiarla después exige reconstruir el sistema.

La guía parte del stage3 `musl-llvm-openrc`, selecciona el perfil `musl/llvm` y
**apila sobre él las características del perfil `hardened`** mediante un perfil
propio. Portage permite esa composición: un perfil puede declarar varios padres
y hereda de todos ellos.

Este camino tiene precedente. En los foros de Gentoo hay quien instaló desde el
tarball musl-clang y añadió después el endurecimiento con un perfil
personalizado. La wiki documenta el apilamiento con esa misma sintaxis para
construir perfiles LLVM de escritorio.

## Qué aporta cada perfil

Conviene saber qué se gana y qué queda pendiente, porque la diferencia es
menor de lo que sugieren los nombres.

Los perfiles 23.0 activan por defecto, **en todos los casos**:

- Ejecutables independientes de posición (PIE)
- Protección de pila `-fstack-protector-strong`
- Protección frente a colisiones de pila
- RELRO y `BIND_NOW`

El perfil `hardened` añade sobre esa base:

| Aporte                              | Origen                                    |
|-------------------------------------|-------------------------------------------|
| `USE="hardened pic xtpax"`          | `features/hardened/make.defaults`         |
| `USE="cet"` en amd64                | `features/hardened/amd64/make.defaults`   |
| `PROFILE_IS_HARDENED=1`             | `features/hardened/make.defaults`         |
| `pie` forzada en `use.force`         | `features/hardened/use.force`             |
| `sys-apps/elfix` en el conjunto base | `features/hardened/packages`              |
| `xattr` forzada en tar, coreutils y portage | `features/hardened/package.use.force` |
| `-D_FORTIFY_SOURCE=3` en vez de 2   | Cadena de herramientas del perfil          |
| Aserciones de la biblioteca estándar | Cadena de herramientas del perfil          |

Las dos últimas filas son las únicas que el apilamiento no resuelve por sí
solo, porque provienen de la configuración de GCC y no de una variable del
perfil. Ambas se recuperan a mano en `make.conf`, como se indica más abajo.

Una observación honesta sobre `xtpax`: la bandera marca binarios con atributos
extendidos de PaX, y el propio perfil establece `PAX_MARKINGS="none"`. Sin un
núcleo con PaX —que ya no se distribuye públicamente— el marcado no cambia
nada en tiempo de ejecución. La guía conserva la bandera porque forma parte del
conjunto que define el perfil y su coste es nulo, no porque aporte una defensa
activa.

## El perfil apilado

Portage necesita que el perfil viva en un repositorio. Crea uno local:

```bash
emerge app-eselect/eselect-repository dev-vcs/git
eselect repository create local
```

Comprueba que `metadata/layout.conf` declare el formato de perfil que admite la
sintaxis `repositorio:ruta`. `eselect repository create` no siempre la escribe:

```bash
cat /var/db/repos/local/metadata/layout.conf
```

```ini
masters = gentoo
profile-formats = portage-2
```

Crea el perfil y declara sus padres:

```bash
mkdir -p /var/db/repos/local/profiles/musl-llvm-hardened
echo 8 > /var/db/repos/local/profiles/musl-llvm-hardened/eapi

cat > /var/db/repos/local/profiles/musl-llvm-hardened/parent <<'EOF'
gentoo:default/linux/amd64/23.0/musl/llvm
gentoo:features/hardened
EOF
```

El orden importa. Portage procesa los padres en secuencia y acumula las
variables incrementales como `USE`, de modo que `features/hardened` aporta sus
banderas sin descartar las del perfil LLVM. Las variables que no son
incrementales —`CC`, `CXX`, `LD` y el resto de las herramientas que define
`features/llvm`— permanecen intactas, porque el perfil `hardened` no las toca.

Registra el perfil para que `eselect` lo muestre:

```bash
echo "$(portageq envvar ARCH) musl-llvm-hardened exp" \
    >> /var/db/repos/local/profiles/profiles.desc
```

Selecciónalo:

```bash
eselect profile list
eselect profile set <número>
eselect profile show
```

Verifica que el apilamiento surtió efecto:

```bash
portageq envvar USE | tr ' ' '\n' | grep -E '^(hardened|pic|xtpax|cet)$'
portageq envvar CC
```

La primera orden debe listar las cuatro banderas; la segunda debe responder
`clang`.

## Fricción conocida

`features/hardened/make.defaults` establece `USE="... -jit -orc"`. En un
sistema construido sobre GCC esas banderas apenas se notan. Aquí desactivan el
compilador en tiempo de ejecución de LLVM y su capa ORC, de la que dependen
algunos consumidores. Mesa es el caso habitual: el renderizador
por software y varias rutas de OpenCL usan el JIT de LLVM.

Si algo que necesitas depende de esas capacidades, reactívalas por paquete en
lugar de desarmar el perfil:

```bash
mkdir -p /etc/portage/package.use
echo "media-libs/mesa llvm" >> /etc/portage/package.use/mesa
```

El resto del sistema conserva `-jit -orc`.

## Configuración de make.conf

```ini
# /etc/portage/make.conf

# --- Cadena de herramientas ---
# El perfil musl/llvm ya define CC, CXX, LD y las utilidades de LLVM.
# Declararlas otra vez aquí sólo duplica lo que el perfil garantiza.

# --- Endurecimiento ---
# El perfil 23.0 aporta PIE, SSP, protección frente a colisiones de pila,
# RELRO y BIND_NOW. Estas dos macros cubren lo que el perfil hardened
# obtiene de la configuración de GCC y que Clang no hereda: el nivel 3 de
# FORTIFY_SOURCE y las aserciones de la biblioteca estándar de C++.
HARDENING_FLAGS="-D_FORTIFY_SOURCE=3"
HARDENING_CXXFLAGS="-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_FAST"

# --- Optimización ---
# ThinLTO es la variante de LTO que recomienda Gentoo para Clang: enlaza
# en paralelo y consume mucha menos memoria que el LTO monolítico.
# Los avisos convertidos en errores detectan incompatibilidades entre
# unidades de traducción que sólo se manifiestan al enlazar.
WARNING_FLAGS="-Werror=odr -Werror=strict-aliasing"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${HARDENING_FLAGS} ${WARNING_FLAGS}"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS} ${HARDENING_CXXFLAGS}"
LDFLAGS="${COMMON_FLAGS} ${LDFLAGS}"

# --- Paralelismo ---
# Sustituye el valor por el que informe 'nproc': make.conf no expande
# órdenes.
MAKEOPTS="-j16 -l16"
EMERGE_DEFAULT_OPTS="--jobs 4 --load-average 16 --keep-going"

# --- USE ---
# 'lto' hace que los paquetes que consultan la bandera construyan con LTO.
# El perfil apilado ya aporta hardened, pic, xtpax y cet.
USE="lto"

# --- Licencias ---
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"

# --- Ramas ---
# El sistema base sigue la rama estable. Los paquetes que requieren la
# rama de pruebas se habilitan uno por uno en package.accept_keywords.
ACCEPT_KEYWORDS=""
```

`-O3` es una elección deliberada de esta guía; Gentoo recomienda `-O2` como
valor general. Si un paquete falla al compilar con `-O3`, baja su nivel de
forma individual con `/etc/portage/env` en lugar de rebajar todo el sistema.

`-ffast-math` no aparece en la configuración global, y no debería. Relaja
garantías de la aritmética de punto flotante que muchos paquetes suponen
vigentes. Aplícalo por paquete si tienes un motivo y has medido el resultado.

## Optimización en tiempo de enlace

Gentoo admite LTO directamente, así que toda la configuración vive en
`make.conf`. El overlay `gentooLTO` está en modo de mantenimiento y su propio
README remite a ese mismo soporte de Gentoo.

Con Clang, la variante correcta es ThinLTO:

```
-flto=thin
```

ThinLTO divide el trabajo por unidad de traducción y lo enlaza en paralelo, lo
que reduce el tiempo y la memoria frente al LTO monolítico. En una compilación
grande la diferencia de memoria decide si la máquina termina o se queda sin
RAM.

Los avisos `-Werror=odr` y `-Werror=strict-aliasing` señalan problemas que sólo
aparecen al enlazar con LTO. Conviene saber que Clang aún no implementa esos
diagnósticos por completo, de modo que su presencia documenta la intención y
protegerá en cuanto el compilador los emita.

Cuando un paquete falle al compilar con LTO, desactívalo sólo para él:

```bash
mkdir -p /etc/portage/env
cat > /etc/portage/env/no-lto <<'EOF'
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,--as-needed"
EOF

echo "categoría/paquete no-lto" >> /etc/portage/package.env/no-lto
```

## Reconstrucción del sistema

Cambiar de perfil y de banderas exige reconstruir todo lo instalado para que el
sistema sea coherente:

```bash
emerge --ask --verbose --update --deep --newuse @world
emerge --ask --emptytree @world
```

La segunda orden reconstruye cada paquete, incluidos los que no cambiaron de
versión. En una máquina de escritorio tarda horas o días según el hardware y el
software instalado. Ejecútala cuando puedas dejar el equipo trabajando.

Comprueba después que el endurecimiento llegó a los binarios:

```bash
emerge app-misc/pax-utils
scanelf -e /usr/bin/emerge
checksec --file=/usr/bin/clang     # app-admin/checksec
```

## El núcleo con Clang

El núcleo se construye con LLVM mediante las variables `LLVM=1` y `LLVM_IAS=1`,
que seleccionan Clang, LLD y el ensamblador integrado.

Asocia ese entorno a las fuentes del núcleo:

```bash
mkdir -p /etc/portage/env /etc/portage/package.env
echo 'KBUILD_BUILD_ENV="LLVM=1 LLVM_IAS=1"' > /etc/portage/env/kernel-clang
echo "sys-kernel/* kernel-clang" >> /etc/portage/package.env/kernel
```

Y compila:

```bash
cd /usr/src/linux
make LLVM=1 LLVM_IAS=1 -j"$(nproc)"
make LLVM=1 LLVM_IAS=1 modules_install
make LLVM=1 LLVM_IAS=1 install
```

Los detalles de configuración del núcleo, del initramfs y del cargador de
arranque están en [installation.md](instalacion.md).

---

<div align="center">

<img src="../../assets/flag-spqr.svg" alt="" height="14"> **[Latina](../la/instrumenta.md)** · <img src="../../assets/flag-burgundy.svg" alt="" height="14"> **Español** · <img src="../../assets/flag-england.svg" alt="" height="14"> **[English](../en/toolchain.md)**

</div>
