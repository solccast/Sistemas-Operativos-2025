# Sistemas Operativos - Teoría - 2

## ¿Qué es el Kernel Linux?
El kernel Linux es un programa que ejecuta programas y gestiona dispositivos de hardware. Es el encargado de que el software y el hardware puedan trabajar juntos. En un sentido estricto, es el Sistema Operativo.
Funciones Principales:
- Administración de memoria principal.
- Administración de uso de la CPU.

Características:
- Es de código abierto a los usuarios (ejemplo: kernel/sched.c).
- Está liberado bajo la licencia GPLv2.
- Soporta todas las arquitecturas en una misma estructura de código fuente.
- Está escrito mayoritariamente en lenguaje C, con partes en Assembler para instrucciones especiales y de bajo nivel, y Rust para implementar módulos (incluido recientemente a partir de la versión 6.1).
- Su desarrollo es colaborativo, con la participación de empresas, universidades y desarrolladores independientes. 

El kernel está organizado en subsistemas, cada uno mantenido por uno o más responsables ("maintainers"). Estos responsables aceptan parches y "pull requests" de los desarrolladores y luego interactúan con Linus Torvalds para incluir las modificaciones en las versiones candidatas. Cada "maintainer" tiene su propio repositorio de desarrollo. 
La división del kernel de GNU/Linux en subsistemas está motivada principalmente por la necesidad de organizar y gestionar su complejidad. Cada subsistema se enfoca en una funcionalidad específica del kernel y es mantenido por uno o más responsables

En cuanto a su arquitectura, el kernel de GNU/Linux se describe como un núcleo monolítico híbrido. Los drivers y el código del kernel se ejecutan en modo privilegiado. Su naturaleza híbrida radica en la posibilidad de cargar y descargar funcionalidad a través de módulos.

## Versionado
### Versionado previo a la versión 2.6:
Se utilizaba un esquema X.Y.Z, donde X indicaba la serie principal, Y si era una versión de producción (par) o desarrollo (impar), y Z los bugfixes.

**Linux 2.6 a 3.0:**
- Fines de 2003: Se lanza la versión 2.6, con mejoras significativas como soporte de threads, mejoras en la planificación y soporte de nuevo hardware.
- 03/08/2011: Se anuncia la versión 2.6.39.4 como la última de su revisión.
- 17/07/2011: Se lanza la versión 3.0. Linus Torvalds justifica el cambio con blabla marketing. Esta versión fue totalmente compatible con el Kernel 2.6 y proporcionó mejoras en virtualización y sistemas de archivos.

El esquema de versionado para Linux >= 2.6 y < 3.0 era A.B.C.[D], donde A era la versión principal, B la revisión mayor, C la revisión menor (nuevos drivers o características), y D correcciones de errores graves.

### Versionado en 3.0 y superiores:
A partir de la versión 3.0, se volvió a un esquema de 3 números: A.B.C[-rcX]. A denota la revisión mayor, B la revisión menor, C el número de revisión y rcX las versiones de prueba.

**Hitos en versiones posteriores:**
- 4.0 (12/04/2015): Permitió aplicar parches y actualizaciones sin reiniciar el sistema y ofreció soporte para nuevas CPUs.
- 5.0 (03/03/2019): Introdujo soporte de energy-aware scheduling, namespaces para binderfs, archivos swap en BTRFS y Kernel Lockdown Mode.
- 5.x: Se destacan el soporte inicial de USB 4 (5.6), un nuevo mecanismo para manejar systemcalls de Windows (5.11), Landlock security module (5.13), una nueva syscall para crear áreas de memoria secretas (5.14) y soporte opcional para paquetes IPv6 de más de 64KB (5.19).
- 6.0 (02/10/2022): Primer release con soporte para módulos escritos en Rust(6.1), mejoras en RAID5/6 de Btrfs (6.2), mejoras de rendimiento y fragmentación para Btrfs (6.3), soporte inicial para USB4.0 v2 (6.5), nuevo scheduler de tareas EEVDF (6.6), soporte para BCachefs (6.7) y optimizaciones de networking (6.8).

## Proceso de compilación de Linux:
Se describe el proceso en los siguientes pasos:
- Obtener el código fuente.
- Preparar el árbol de archivos del código fuente.
- Configurar el kernel.
- Construir el kernel e instalar los módulos.
- Reubicar el kernel.
- Creación del initramfs.
- Configurar y ejecutar el gestor de arranque (GRUB).
- Reiniciar y probar el nuevo kernel.