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

## Práctica guiada

1. Código de `my_sys_call.c`
● **Para qué sirven los macros SYS_CALL_DEFINE?**
Se utiliza para definir nuevas llamadas al sistema dentro del kernel. Estos macros generan la función correcta con el nombre esperado por el sistema para poder invocarla desde espacio de usuario. 

● **¿Para que se utilizan la macros `for_each_process` y `for_each_thread`?**
- `for_each_process`: se utiliza para recorrer todos los procesos del sistema. Itera sobre cada estructura `task_struct` que representa un proceso en el scheduler del kernel. 
- `for_each_thread`: recorre todos los hilos de un proceso específico. Permite iterar sobre los threads que comparten el mismo `thread group`.

● **¿Para que se utiliza la función copy_to_user?**
Se usa para copiar datos desde el espacio del kernel hacia el espacio de usuario dado que ambos ocupan manejan espacios de memoria separados entonces no se puede acceder directamente a punteros del usuario desde el kernel. 

● **¿Para qué se utiliza la función printk?, ¿porque no la típica printf?**
`printk` es la función que se usa en el kernel de Linux para imprimir mensajes de depuración o información al log del sistema. No se puede usar `printf` en el kernel porque `printf` pertenece al espacio de usuario (por ejemplo, lo usa `glibc`).

● **¿Podría explicar que hacen las sytem call que hemos incluido?**
- `my_sys_call(int arg)`:Es una syscall de prueba que simplemente recibe un número y lo imprime en el log del kernel usando printk.

- `get_task_info(char __user *buffer, size_t length)`: Recorre todos los procesos activos del sistema y guarda su información (PID, nombre, estado) en un buffer del kernel. Luego, copia ese contenido al espacio de usuario para que pueda ser leído por una aplicación. Es útil para obtener una lista de procesos en ejecución desde el usuario, de forma similar a ps.

- `get_threads_info(char __user *buffer, size_t length)`: Hace algo similar a la anterior, pero además de los procesos, también lista todos los hilos (threads) de cada proceso. Esto proporciona una vista más completa del estado del sistema, incluyendo los hilos de cada proceso.

2. Modificaremos uno de los archivos Makefile del código del Kernel para indicar la compilación de nuestro código agregado en el paso anterior: 
`kernel/Makefile`
```c
obj-y = fork.o exec_domain.o panic.o \
    cpu.o exit.o softirq.o resource.o \
    sysctl.o capability.o ptrace.o user.o \
    signal.o sys.o umh.o workqueue.o pid.o task_work.o \    
    extable.o params.o \
    kthread.o sys_ni.o nsproxy.o \
    notifier.o ksysfs.o cred.o reboot.o \
    async.o range.o smpboot.o ucount.o regset.o \
    my_sys_call.o
```

3. Añadir una entrada al final de la tabla que contiene todas las Syscalls. 
![alt text](image-2.png)

Agregar los headers al vector manejador de System calls:

![alt text](image-3.png)

4. Compilar el kernel de nuevo. 
Se ejecutó mediante 
```bash
make -jX
make modules_install
make install
```

5. Verificación de las system calls que ya sean parte del kernel 
```bash
grep get_task_info "/boot/System.map-$(uname -r)"
```
![alt text](image-4.png)

6. Creación del script que pone a prueba la ejecución de la System Call 
![alt text](image-5.png)

[^1]: https://www.ibm.com/docs/es/aix/7.3?topic=concepts-kernel-environment#kernextc_kern_env