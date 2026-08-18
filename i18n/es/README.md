<!--
i18n/es/README.md
@fraxgut
CC-BY-SA-4.0
Spanish landing page and index for the installation guide
-->

# Guía avanzada de instalación de Gentoo Linux

> 🌐 **Idioma:** [English](../en/README.md) · **Español**

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
## Licencia

Salvo indicación en contrario, esta documentación se distribuye bajo la licencia
Creative Commons Atribución-CompartirIgual 4.0 Internacional (CC BY-SA 4.0).
Consulta [LICENCE.md](LICENCE.md).

El nombre y el logotipo de Gentoo pertenecen a la Gentoo Foundation y quedan
fuera de esta licencia.

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

