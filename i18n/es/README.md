<!--
i18n/es/README.md
@fraxgut
CC-BY-SA-4.0
Spanish landing page and index for the installation guide
-->

# Guía avanzada de instalación de Gentoo Linux

> 🌐 **Idioma:** [Latina](../la/README.md) · **Español** · [English](../en/README.md)

Instalación de Gentoo Linux en AMD64 sobre musl, con OpenRC, LUKS2, Btrfs,
LLVM/Clang, LTO y el núcleo Zen.

## Sobre esta guía

Esta guía documenta una configuración concreta, no una instalación general de
Gentoo. Cada pieza responde a una decisión explícita, y el texto explica el
motivo además del procedimiento.

Está dirigida a personas con experiencia previa en Gentoo. Da por supuestos el
manejo de `emerge`, las banderas USE, los perfiles y la configuración manual
del núcleo.

## Configuración objetivo

| Componente              | Elección                                          |
|-------------------------|---------------------------------------------------|
| Arquitectura            | AMD64                                             |
| Biblioteca C            | musl                                              |
| Sistema de inicio       | OpenRC                                            |
| Cifrado                 | LUKS2 con Argon2id                                |
| Sistema de archivos     | Btrfs con subvolúmenes                            |
| Cadena de herramientas  | Clang, LLD y libc++                               |
| Optimización            | ThinLTO                                           |
| Núcleo                  | Zen                                               |
| Endurecimiento          | Perfil `hardened` apilado sobre `musl/llvm`       |
| Volúmenes lógicos       | LVM como variante opcional                        |

## Documentación

1. [Instalación](instalacion.md) — del medio de instalación al primer arranque.
2. [Arquitectura de almacenamiento](almacenamiento.md) — particionado, LUKS2, Btrfs, compresión, TRIM e intercambio.
3. [Cadena de herramientas LLVM y LTO](herramientas.md) — el perfil apilado, el endurecimiento y la optimización.
4. [Optimización](optimizacion.md) — los niveles, las puertas por hardware y las vías de escape.
5. [Solución de problemas](problemas.md) — recuperación por síntoma.

## Advertencias

Los perfiles `musl/llvm` y `musl/hardened` están marcados como experimentales
en el árbol de Gentoo. El software que compila con GCC y glibc no siempre
compila con Clang y musl.

La biblioteca C, la cadena de herramientas y el sistema de inicio quedan
fijados al desempaquetar el stage3. Cambiarlos después exige reconstruir el
sistema, así que la elección se hace antes de instalar.

Los procedimientos de particionado destruyen datos. Haz copias de seguridad y
comprueba dos veces el nombre del dispositivo.

## Hoja de ruta

- [x] Btrfs sobre LUKS2 con subvolúmenes
- [x] musl con OpenRC
- [x] Clang, LLD y libc++ desde el stage3 oficial
- [x] Endurecimiento apilado sobre el perfil LLVM
- [x] ThinLTO con el soporte de Gentoo
- [x] Núcleo Zen construido con LLVM
- [x] Intercambio nativo de Btrfs con hibernación
- [ ] Secure Boot, para autenticar la cadena de arranque que queda fuera de LUKS
- [ ] SELinux
- [ ] Configuración del núcleo de ejemplo
- [ ] Instantáneas automáticas y arranque desde instantánea

## Licencia

Salvo indicación en contrario, esta documentación se distribuye bajo la
licencia Creative Commons Atribución-CompartirIgual 4.0 Internacional
(CC BY-SA 4.0). Consulta [LICENCE.md](../../LICENCE.md).

El nombre y el logotipo de Gentoo pertenecen a la Gentoo Foundation y quedan
fuera de esta licencia. Su uso se rige por las [Gentoo Name and Logo Usage
Guidelines](https://www.gentoo.org/inside-gentoo/foundation/name-logo-guidelines.html).

Esta guía es un proyecto personal. Habla únicamente por su autor, y la Gentoo
Foundation no participa en ella.

## Contribuir

Las contribuciones son bienvenidas. [CONTRIBUTING.md](../../CONTRIBUTING.md)
describe las convenciones de commits y cómo mantener sincronizadas las
traducciones.

## Contacto

Franco Gutiérrez — [@fraxgut](https://github.com/fraxgut) — franco.gutierrez.1@ug.uchile.cl

## Referencias

- [Manual de Gentoo para AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Proyecto musl de Gentoo](https://wiki.gentoo.org/wiki/Project:Musl)
- [Perfiles personalizados de Portage](https://wiki.gentoo.org/wiki/Portage/Profiles/Custom_profiles)
- [LLVM/Clang en Gentoo](https://wiki.gentoo.org/wiki/LLVM)
- [LTO en Gentoo](https://wiki.gentoo.org/wiki/LTO)
- [Documentación de Btrfs](https://btrfs.readthedocs.io/)
- [Cifrado de disco completo con dm-crypt](https://wiki.gentoo.org/wiki/Dm-crypt_full_disk_encryption)
- [Dracut en Gentoo](https://wiki.gentoo.org/wiki/Dracut)

---

> 🌐 **Idioma:** [Latina](../la/README.md) · **Español** · [English](../en/README.md)
