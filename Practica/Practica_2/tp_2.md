# Sistemas Operativos - Práctica 2 [2025]

## System Calls 

### Conceptos generales 
1. ¿Qué es una System Call? ¿Para qué se utiliza?
Una llamada al sistema es una rutina que permite a una aplicación de usuario solicitar acciones que requieren privilegios especiales. 
Las funciones del sistema operativo se ponen a disposición del programa de aplicación en forma de bibliotecas de programación. Un conjunto de funciones de biblioteca que se encuentran en una biblioteca como, por ejemplo, `libc.a` puede tener funciones que realizan algún proceso en modalidad de usuario y, a continuación, inician internamente una llamada al sistema.
Las funciones de sistema operativo disponibles para los programas de aplicación se pueden dividir o mover entre funciones de modalidad de usuario y funciones de modalidad de kernel según sea necesario para diferentes releases o plataformas de máquina. [^1]
 
2. ¿Para qué sirve la macro syscall? Describa el propósito de cada uno de sus parámetros
`syscall` es una función proporcionada por `glibc` para hacer system calls de forma explícita. Es más difícil de usar y menos portátil que usar las librerías (`libc`) pero aún así es ás fácil y portátil que codificar instrucciones en assembler, syscall es más útil cuando se trabaja con system calls que son especiales o más nuevas que la librería que se está usando. 

**Parámetros:**
- **Número de la system call:** indica qué system call específica se desea realizar. Es único. 
- Los parámetros restantes son parámetros de para la system call, en orden, y sus significados dependen de la system call realizada. Se permiten hasta 5. Los que están de más se ignoran. 

3. Ejecute el siguiente comando e identifique el propósito de cada uno de los archivos que encuentra 
```bash
ls -lh /boot | grep vmlinuz
```
![alt text](image.png)
Lo que se muestra son las imágenes del núcleo del kernel de Linux que están instaladas en el sistema. 
Su nombre indica la versión del kernel. 
Estos archivos son usados por el bootloader (GRUB) al arrancar el sistema donde carga uno de estos núcleos en memoria y lo ejecuta. 

4. Acceda al codigo fuente de GNU Linux, sea visitando https://kernel.org/ o bien trayendo el código del kernel(cuidado, como todo software monolítico son unos cuantos gigas)
```bash
git clone  https://github.com/torvalds/linux.git
```
5. ¿Para qué sirve el siguiente archivo `arch/x86/entry/syscalls/syscall_64.tbl`?
Se encarga de definir la tabla de llamadas al sistema para la arquitectura `x86_64`. Asocia cada número de **syscall** con:
- el nombre del syscall
- su función implementada en el kernel
- y el ABI (applicaction binary interface): common, 64, x32, etc.
Este archivo es usado por el kernel durante la compilación para generar el código que mapea llamadas al sistema (por ejemplo, cuando un programa hace syscall 0 se invoca __x64_sys_read).

![alt text](image-1.png)

6. ¿Para qué sirve la herramienta `strace`?¿Cómo se usa?
Es una herramienta para depurar, analizar y entender lo que hace un programa en Linux a nivel del sistema operativo.
Permite ver en tiepo real las **system calls* que realiza un programa. 
Se usa de la siguiente forma: 
- `strace ./mi_programa`: se verán todas las llamadas que realiza el programa `mi_programa`.
- `strace -e openat ./mi_programa`: para ver todas las systemcalls `open` que se realizan desde `mi_programa`.
- `strace -o salida.txt ./mi_programa`: para almacenar la salida en un archivo. 
- `strace -p <PID>`: adjuntarse a un proceso ya en ejecución. 

7. ¿Para qué sirve la herramienta `ausyscall`? ¿Cómo se usa?
`ausyscall` forma parte del sistema de auditoría de Linux. Permite:
- Consultar el número de syscall a partir de su nombre,
- consultar el nombre de syscall a partir de número,
- usarlo para escribir reglas de auditoría con `auditctl` o analizar logs de `auditd`.



[^1]: https://www.ibm.com/docs/es/aix/7.3?topic=concepts-kernel-environment#kernextc_kern_env