<!--
i18n/es/optimisation.md
@fraxgut
CC-BY-SA-4.0
Optimisation tiers, hardware gates and the escape hatches that keep them safe
-->

# Optimización

> 🌐 **Idioma:** [English](../en/optimisation.md) · **Español**

Esta guía optimiza con fuerza, y lo hace dentro de un orden de prioridades
fijo:

```
SEGURIDAD  >  ENDURECIMIENTO  >  OPTIMIZACIÓN  >>>  CONVENCIONALIDAD
```

Los tres primeros términos compiten de verdad, y el orden decide quién cede.
El cuarto está muy atrás: que Gentoo recomiende `-O2` no es motivo para usar
`-O2` si medimos algo mejor. Que una bandera sea poco habitual tampoco la
descarta.

## Índice

1. [Lo que esta guía mantiene encendido](#lo-que-esta-guía-mantiene-encendido)
2. [Los niveles](#los-niveles)
3. [Nivel 0: seguridad](#nivel-0-seguridad)
4. [Nivel 1: la base](#nivel-1-la-base)
5. [Nivel 2: agresivo](#nivel-2-agresivo)
6. [Nivel 3: por paquete](#nivel-3-por-paquete)
7. [Puertas por hardware](#puertas-por-hardware)
8. [El asignador de memoria](#el-asignador-de-memoria)
9. [Las vías de escape](#las-vías-de-escape)
10. [El núcleo conservador](#el-núcleo-conservador)
11. [Lo que se rompe](#lo-que-se-rompe)
12. [Cómo medir](#cómo-medir)

## Lo que esta guía mantiene encendido

Las configuraciones agresivas que circulan por foros suelen comprar velocidad
apagando defensas. Un ejemplo conocido reúne `-hardened -ssp -seccomp -pie
-pic` en la variable USE y añade un bloque de banderas dedicado a deshacer el
endurecimiento: `-U_FORTIFY_SOURCE`, `-fno-stack-protector`,
`-fno-stack-clash-protection`, `-fcf-protection=none`.

Esta guía toma de esas configuraciones las optimizaciones y conserva las
defensas:

| Defensa                        | Estado aquí | Qué cuesta mantenerla                     |
|--------------------------------|-------------|-------------------------------------------|
| `USE="hardened pic"`           | Encendida   | Casi nada en la práctica.                 |
| PIE y `-fPIC`                  | Encendidas  | Un registro en x86-64. Medible con esfuerzo. |
| `-fstack-protector-strong`     | Encendida   | Un prólogo por función con arreglos.      |
| `-fstack-clash-protection`     | Encendida   | Sondeos de pila en funciones grandes.     |
| `-D_FORTIFY_SOURCE=3`          | Encendida   | Comprobaciones de tamaño que el compilador resuelve en tiempo de compilación cuando puede. |
| `-fcf-protection=full`, `USE="cet"` | Encendidas | Instrucciones ENDBR. Ruido en el tamaño. |
| `USE="seccomp"`                | Encendida   | Prácticamente nulo.                       |
| Aserciones de libc++           | Encendidas  | Comprobaciones de precondición en contenedores. |

Hay un argumento empírico a favor de esta elección, y viene del mismo hilo que
propone el desendurecimiento. Uno de sus participantes midió las banderas una
por una con Phoronix Test Suite. Buena parte de los resultados quedó dentro del
margen de error. Sobre `_FORTIFY_SOURCE` escribió que volvía a encenderlo si no
encontraba diferencia, y que ése fue el caso. Añadió
además un ejemplo concreto de un programa donde `_FORTIFY_SOURCE` frenaba
fugas de memoria y desbordamientos de montículo reales.

Otro participante señaló algo aún más incómodo para el desendurecimiento: las
banderas USE `-pie -pic` sólo afectan a los paquetes que las declaran, así que
el resto del sistema se sigue compilando con `-fPIC` de todos modos. La
defensa se pierde a medias y la velocidad no aparece.

Una corrección de nomenclatura, por si llegas desde esos hilos:
**`extra-hardened` no es una bandera USE global**. En el árbol de Gentoo
existe únicamente en `app-admin/clsync`, y allí significa lo contrario de lo
que sugiere su nombre en aquel contexto: activa comprobaciones adicionales de
seguridad a costa de rendimiento.

## Los niveles

La configuración se organiza en cuatro niveles. Cuando algo falla, se relaja
desde el número más alto hacia abajo, nunca al revés.

| Nivel  | Contenido                                     | Se relaja  |
|--------|-----------------------------------------------|------------|
| **0**  | Seguridad: firmas, seccomp, capacidades       | Nunca      |
| **1**  | Base: `-O3`, `-march=native`, ThinLTO         | El último  |
| **2**  | Agresivo: matemática rápida, desenrollado, Polly | El primero |
| **3**  | Por paquete: PGO, asignadores, jumbo-build    | Individual |

## Nivel 0: seguridad

Este nivel no negocia con el rendimiento porque no compite con él.

```ini
USE="verify-sig seccomp caps filecaps"
```

`verify-sig` comprueba las firmas de los archivos que Portage descarga. Es la
defensa más barata del sistema: mueve la confianza desde «el espejo me entregó
algo» hasta «el desarrollador firmó esto».

`seccomp` filtra llamadas al sistema en tiempo de ejecución. Su coste es
prácticamente nulo en la enorme mayoría de los casos, y apagarlo tiene
consecuencias serias: en contenedores elimina la mayor parte del aislamiento.

`caps` y `filecaps` reemplazan binarios con bit setuid por capacidades
concretas. Un programa que sólo necesita abrir un puerto bajo recibe esa
capacidad en lugar de convertirse en root entero.

Añade también la verificación de firmas para el propio árbol de Portage:

```ini
FEATURES="${FEATURES} webrsync-gpg"
```

## Nivel 1: la base

Se aplica a todo el sistema.

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
```

`-O3` frente a `-O2` es una elección deliberada. Desde que `-ftree-vectorize`
pasó de `-O3` a `-O2`, la diferencia entre ambos se estrechó bastante, y en
muchos programas cae dentro del ruido. Mantenemos `-O3` porque el coste es
tiempo de compilación, y el tiempo de compilación es la moneda que esta
instalación ya decidió gastar.

`-march=native` describe el procesador con más detalle que cualquier nombre de
familia.

ThinLTO reparte el trabajo por unidad de traducción y enlaza en paralelo. Hay
un matiz honesto: el LTO monolítico produce código algo mejor, y ThinLTO
compra paralelismo y memoria a cambio de esa diferencia. En una máquina que
compila LibreOffice, esa memoria decide si el enlace termina.

## Nivel 2: agresivo

Aquí empieza lo que se relaja primero.

```ini
# Matemática rápida sin perder NaN ni infinitos
FASTMATH_FLAGS="-ffast-math -fno-finite-math-only"

# Diseño de código
FUN_FLAGS="-funroll-loops -fno-plt -fno-semantic-interposition \
           -fdata-sections -ffunction-sections \
           -falign-functions=${CACHELINE}"

# Polly, el equivalente de Graphite en LLVM
POLLY_FLAGS="-mllvm=-polly -mllvm=-polly-vectorizer=stripmine \
             -mllvm=-polly-invariant-load-hoisting"

COMMON_FLAGS="-O3 -pipe -march=native -flto=thin ${FASTMATH_FLAGS} ${FUN_FLAGS}"
LDFLAGS="${COMMON_FLAGS} -Wl,-O2 -Wl,--as-needed -Wl,--gc-sections \
         -Wl,-z,relro -Wl,-z,now -Wl,-z,pack-relative-relocs"
```

Cuatro observaciones sobre estas banderas.

**`-ffast-math` en lugar de `-Ofast`.** Clang marcó `-Ofast` como obsoleto y lo
sustituye por exactamente esta expansión. Como el sistema de esta guía compila
con Clang, escribimos la forma que Clang mantiene.

**`-fno-finite-math-only` es lo que hace tolerable a `-ffast-math`.** Restituye
el tratamiento de NaN e infinitos, que es la fuente habitual de roturas. Aun
así, `-ffast-math` autoriza reasociación: si un programa tuyo depende del
resultado exacto en coma flotante, mídelo antes de confiar en él.

**`-funroll-loops` es sensible al código concreto.** Depende de la geometría de
caché, de las mitigaciones activas del núcleo y del programa. Mejora unos
casos y empeora otros, así que entra en el nivel que se mide, no en el que se
asume.

**Polly usa la sintaxis `-mllvm=`, con signo igual.** La forma separada
`-mllvm -polly` rompe la compilación de algunos paquetes grandes. Polly llega
a través de `llvm-runtimes/clang-runtime` con `USE="polly"`, y la propia
descripción de la bandera advierte que después hay que pasar `-mllvm -polly`
para usarlo. No lo actives en todos los paquetes: varios fallan.

`-fno-semantic-interposition` merece una nota. Optimiza código PIE, así que
sirve precisamente porque este sistema sí construye ejecutables independientes
de posición. En un sistema desendurecido sin PIE no haría nada.

Graphite queda fuera: es de GCC y no existe en Clang. Sólo aplicaría a los
paquetes que caigan a la vía de escape con GCC.

## Nivel 3: por paquete

Estas banderas no son globales, y el árbol acota bastante dónde existen.

| Bandera USE   | Paquetes que la ofrecen | Qué hace                                        |
|---------------|-------------------------|-------------------------------------------------|
| `pgo`         | 11                      | Compila, ejecuta un perfil y recompila.         |
| `jumbo-build` | 5                       | Acelera la **compilación**, no el binario.      |
| `native-extensions` | 14                | Compila extensiones nativas en vez de código puro. |
| `custom-cflags` | 1                     | Impide que el paquete descarte tus CFLAGS.      |

`pgo` está en `bash`, `python`, `xz-utils`, `binutils`, `gcc`, `firefox`,
`thunderbird`, `chromium`, `svt-av1` y un par más. Es la optimización con
mejor relación entre ganancia y riesgo de toda esta página, porque mide el
programa real en vez de adivinar. Cuesta duplicar el tiempo de compilación.

```bash
echo "dev-lang/python pgo" >> /etc/portage/package.use/optimisation
echo "app-shells/bash pgo"  >> /etc/portage/package.use/optimisation
echo "app-arch/xz-utils pgo" >> /etc/portage/package.use/optimisation
```

`jumbo-build` combina archivos fuente para compilar más rápido y consume más
memoria. No cambia el binario resultante, así que su lugar está en la lista de
comodidades, no en la de rendimiento.

**BOLT no aparece como bandera USE en el árbol.** Si lo quieres, es trabajo
manual sobre el binario ya enlazado.

## Puertas por hardware

Varias banderas dependen del procesador, y la respuesta se consulta en la
máquina en lugar de copiarse de un foro.

### Alineación de funciones

`-falign-functions=32` se hizo popular por Clear Linux, y ese número sólo
resulta ventajoso en procesadores Intel desde Sandy Bridge. La recomendación
correcta es alinear al tamaño de la línea de caché de instrucciones, que el
sistema informa:

```bash
getconf -a | grep LEVEL1_ICACHE_LINESIZE
```

```
LEVEL1_ICACHE_LINESIZE              64
```

En ese equipo la respuesta es `-falign-functions=64`. Léelo antes de elegir:

```bash
export CACHELINE=$(getconf LEVEL1_ICACHE_LINESIZE)
echo "-falign-functions=${CACHELINE}"
```

### Qué significa `-march=native` en tu equipo

Conviene saber qué activa realmente, sobre todo si vas a compilar para otra
máquina:

```bash
gcc -march=native -E -v - </dev/null 2>&1 | grep cc1
clang -march=native -E -v - </dev/null 2>&1 | grep cc1
```

`app-misc/resolve-march-native` presenta lo mismo de forma legible. El
resultado incluye los tamaños de caché que el compilador usa para decidir, y
sirve para reproducir la configuración en un equipo distinto.

### Banderas de CPU para Portage

```bash
emerge app-portage/cpuid2cpuflags
echo "*/* $(cpuid2cpuflags)" >> /etc/portage/package.use/00cpu-flags
```

## El asignador de memoria

Este punto importa más aquí que en una instalación con glibc, y merece
atención precisamente porque el sistema usa musl.

El asignador de musl prioriza el tamaño del código y la resistencia a la
fragmentación por encima de la velocidad bruta. En cargas que asignan y
liberan mucha memoria, un asignador alternativo rinde bastante más. La
observación circula entre quienes comparan asignadores: jemalloc suele superar
al de glibc, y la diferencia frente al de musl es mayor todavía.

Gentoo ofrece la elección por paquete:

```ini
jemalloc  - Use dev-libs/jemalloc for memory management
tcmalloc  - Use the dev-util/google-perftools libraries to replace malloc()
```

Casi ningún paquete acepta ambas, así que se elige una por paquete:

```bash
echo "dev-db/redis jemalloc" >> /etc/portage/package.use/allocator
```

mimalloc también existe en el árbol para algunos paquetes. Un aviso concreto
de quien lo usa a nivel de enlazador: rompe `bash`. Si lo pruebas, hazlo por
paquete y no a través de `LDFLAGS` globales.

## Las vías de escape

Un sistema optimizado necesita una forma ordenada de rendirse en casos
concretos. `/etc/portage/env` guarda perfiles y `package.env` los asigna.

Estos cuatro cubren casi todo:

```bash
mkdir -p /etc/portage/env
```

`/etc/portage/env/safest.conf` — la rendición completa, para lo que no compila
de ninguna otra forma:

```ini
CC="gcc"
CXX="g++"
COMMON_FLAGS="-O2 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-fuse-ld=bfd -Wl,-O1 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-lto.conf` — conserva las optimizaciones y suelta LTO:

```ini
COMMON_FLAGS="-O3 -pipe -march=native"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,-O2 -Wl,--as-needed"
USE="-lto"
```

`/etc/portage/env/no-fastmath.conf` — para los paquetes que rechazan la
matemática rápida:

```ini
COMMON_FLAGS="-O3 -pipe -march=native -flto=thin"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
```

`/etc/portage/env/heavy.conf` — para los enlaces que agotan la memoria:

```ini
MAKEOPTS="-j1 -l2"
NINJAOPTS="-j1 -l2"
```

Y se asignan así:

```bash
mkdir -p /etc/portage/package.env
echo "dev-lang/python no-fastmath.conf" >> /etc/portage/package.env/10-toolchain
echo "dev-qt/qtwebengine heavy.conf"    >> /etc/portage/package.env/20-desktop
```

Un error frecuente al escribir estos archivos: si defines `COMMON_FLAGS` pero
luego asignas `CFLAGS` a partir de otra variable, el paquete acaba compilando
sin `-march` ni `-O`. Comprueba lo que realmente recibe:

```bash
emerge --info categoría/paquete | grep -E '^(CFLAGS|CXXFLAGS|LDFLAGS)'
```

## El núcleo conservador

La experiencia de quienes mantienen sistemas así coincide en qué capas se
rompen primero: la cadena de herramientas, la biblioteca C, las bibliotecas
gráficas de bajo nivel y las de criptografía.

Un punto de partida razonable, que mantiene el sistema arrancando mientras el
resto se optimiza:

```
# Cadena de herramientas y biblioteca C
sys-devel/binutils      safest.conf
sys-devel/gcc           safest.conf
sys-libs/musl           safest.conf
sys-kernel/linux-headers safest.conf
llvm-core/clang         safest.conf heavy.conf
llvm-core/llvm          safest.conf heavy.conf

# Criptografía y red
dev-libs/openssl        safest.conf
net-misc/curl           safest.conf
sys-libs/zlib           safest.conf

# Gráficos
media-libs/mesa         safest.conf
x11-libs/libX11         safest.conf
x11-libs/libxcb         safest.conf
media-libs/freetype     safest.conf
media-libs/harfbuzz     safest.conf
```

La lógica es de riesgo, no de rendimiento. Un fallo sutil en `openssl` o en
`musl` no se manifiesta como un error de compilación, sino como datos
corruptos o como una comprobación criptográfica que pasa cuando no debería.
Esas piezas se benefician poco de la optimización agresiva y concentran casi
todo el daño posible.

## Lo que se rompe

Lista concreta, recogida de quienes ya lo intentaron.

**No compilan con matemática rápida:**

```
dev-lang/python
net-libs/nodejs
```

**Rechazan explícitamente `-ffinite-math-only`:**

```
sys-apps/systemd-utils
sys-auth/polkit
media-libs/opus
dev-lang/duktape
sys-auth/elogind
```

Estos avisan durante la compilación, lo que es la forma amable de fallar. El
caso preocupante es el contrario: un paquete que acepta las banderas y produce
un binario que falla más tarde con un fallo de segmentación. Activar la suite
de pruebas ayuda a encontrarlos antes:

```ini
FEATURES="${FEATURES} test"
```

Aun así no los detecta todos, y conviene saberlo antes de decidir hasta dónde
llevar el nivel 2.

## Cómo medir

Una configuración agresiva sin medición es una creencia. El protocolo que hace
falta es modesto pero estricto.

```bash
emerge app-benchmarks/hyperfine
```

Compara una carga real, no un bucle sintético:

```bash
hyperfine --warmup 3 --runs 9 \
  'xz -9 -T1 -c /usr/portage/distfiles/muestra.tar > /dev/null'
```

Tres reglas que evitan conclusiones falsas:

**Nueve ejecuciones como mínimo.** Una comparación de tres ejecuciones fabrica
mejoras del diez por ciento que desaparecen con más muestras.

**Sepa qué limita la carga antes de prometer una mejora.** El análisis de
texto, la persecución de punteros y la comparación de cadenas no ganan nada
con vectores más anchos. La aritmética densa en coma flotante sí.

**Informa el resultado honesto**, incluida la respuesta «estas banderas no
compraron nada aquí». Es la respuesta más frecuente, y saberlo ahorra
recompilaciones.

Mide también el otro lado de la balanza: el tiempo de compilación es el precio
que estas banderas cobran, y `genlop` lo lee del registro de Portage.

```bash
emerge app-portage/genlop
genlop -t categoría/paquete
```

---

> 🌐 **Idioma:** [English](../en/optimisation.md) · **Español**
