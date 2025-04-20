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

### Práctica guiada

#### Agregamos una nueva System Call

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

**Makefile**:
 ```bash
        init: prueba_syscall.c
	        gcc -o prueba_syscall prueba_syscall.c 
        run: prueba_syscall
	        ./prueba_syscall
        clean:
	        rm -f prueba_syscall 
```

#### Monitoreando System Calls 
1. Ejecuta el programa anteriormente compilado 
```
./get_task_info
```
2. Ejecute 
```
sudo dmesg
```
![alt text](image-6.png)
![alt text](image-7.png)

`dmesg` muestra mensajes del kernel de Linux, del ring buffer (buffer circular: estructura de datos que tiene un tamaño fijo, cuando llega al final, vuelve al inicio y empieza a sobreescribir los datos más antiguos).
El output son mensajes generados por el kernel. Cada línea normalmente tiene este formato:
```cpp
[ 123.456789] usb 1-1: new high-speed USB device number 4 using ehci-pci
```
Explicación:
- `[ 123.456789]` → Tiempo desde que se encendió el sistema (uptime en segundos).
- `usb 1-1`: → El componente del sistema que generó el mensaje (en este caso, el controlador USB).
- `new high-speed USB device... `→ Descripción del evento (acá, detectó un nuevo dispositivo USB).

3. Ejecute el programa anterior con la siguiente herramienta:`
```bash
#En mi caso el programa se llama de esta forma 
strace ./prueba_syscall
```
La salida se encuentra en [salida.txt](https://github.com/solccast/kernel-vm-personal/commit/3f07c7be0ad03a59e7d0c42b40b7e5a0c202464d) , línea 30. Al ejecutar el echo nos sale el número de system call que se ejecutó 
![alt text](image-8.png)


## Módulos y Drivers 

### Conceptos generales

1. ¿Cómo se denomina en GNU/Linux a la porción de código que se agrega al kernel en tiempo de ejecución? ¿Es necesario reiniciar el sistema al cargarlo? Si no se pudiera utilizar esto ¿cómo deberíamos hacer para proveer la misma funcionalidad en GNU/Linux?
Se denomina **módulo del kernel** o **kernel Module**. Suelen tener extensión `.ko` (kernel object). 
Pueden cargarse y descargarse en tiempo de ejecución. 
```bash
sudo insmod mi_modulo.ko     # Carga el módulo
sudo rmmod mi_modulo         # Lo descarga
```
En caso de que el kernel no admitiera módulos (monolítico sin soporte de carga dinámica) la forma de agregar más funcionalidad sería así: 
    1. Agregar el código directamente al kernel fuente.
    2. Recompilar el kernel completo.
    3. Reiniciar el sistema. 

2. ¿Qué es un driver? ¿Para qué se utiliza?
Un **driver** es un módulo o programa que le dice al sistema operativo cómo interactuar con un hardware específico. Sirve para que el sistema pueda usar hardware de forma genérica, sin tener que saber cómo funciona internamente. 
Se pueden cargar con `modprobe`. 
Aparecen en `/lib/modules$(uname -r)/`

3. ¿Por qué es necesario escribir drivers?
Dado que cada dispositivo es diferente y tiene su propia forma de funcionar el sistema operativo no puede incluir soporte nativo para todos los dispositivos. A parte, favorece al aislamiento y modularidad ya que separan el código del sistema operativo del manejo específico del hardware. 
Un drive generalmente está implementado como módulo del kernel pero también puede formar parte del núcleo desde el arranque (incluido directamente en el archivo `vmlinuz`)

4. ¿Cuál es la relación entre módulo y driver en GNU/Linux?
Un driver es el software que permite al sistema operativo comunicarse con un dispositivo (como una impresora, tarjeta de red, USB, etc). 
Un módulo es una forma de cargar ese driver en el kernel dinámicamente, en tiempo de ejecución.
Entonces:  Un driver puede estar implementado como un módulo del kernel.
5. ¿Qué implicancias puede tener un bug en un driver o módulo?
Debe tenerse en cuenta un driver corre en el espacio del **modo privilegiado** o **modo kernel** por lo tanto no hay protección de memoria, tienen acceso total a todos los recursos del sistema. 
Puede implicar:
- Bloqueo del sistema (kernel panic)
- Caída del rendimiento
- Corrupción de meoria o datos
- Vulnerabilidades de seguridad 

6. ¿Qué tipos de drivers existen en GNU/Linux?

|Tipos de drivers| Detalle|
| -- | -- | 
| de dispositivos (device drivers) | controlan dispositivos físicos | 
| de pseudo-dispositivos (virtuales) | emulan dispositivos. Ej: `/dev/null` descarta todo lo que se escribe en él; `/dev/zero` devuelve una secuencia infinita de ceros; `ramdisk` usa memoria RAM como si fuera un disco. |
| de sistema de archivos | manejan como se leen/escriben datos en distintos sistemas de archivos | 
| de bus | controlan como se comunican los dispositivos conectados mediante buses. Ej: `usbcore` para USB; `pci_generic` para PCI. |
| de virtualización / hypervisores | permiten que Linux corra sobre o dentro de máquinas virtuales | 
| de energía / sensores | controlan funciones como la administración de energía, de ventiladores, de temperatura. |

7. ¿Qué hay en el directorio /dev? ¿Qué tipos de archivo encontramos en esa ubicación?
Contiene archivos de dispositivos. Son interfaces entre el sistema operativo y los dispositivos. 
Hay dos tipos principales de archivos de dispositivo: 
- **Archivos de dispositivo de carácter (chardevice) - c**: transfieren datos byte a byte (como un flujo). Ej: teclados, mouse, puertos serie.
- **Archivos de dispositivo de bloque (block device) - b**: transfieren datos por bloques con acceso aleatorio. Ej: discos, pendrives.


8. ¿Para qué sirven el archivo `/lib/modules/<version>/modules.dep` utilizado por el comando `modprobe`?
Es un archivo de dependneicas que mantiene una listade qué módulos del kernel dependen de otros. 
Cuando se quiere cargar un módulo con: 
```bash
modprobe nombre_modulo
```
`modprobe`: consulta `modules.dep`, ve si ese módulo necesita otros módulos para funcionar y en caso de que así sea, carga todas las dependencias en el orden correcto.  

9. ¿En qué momento/s se genera o actualiza un `initramfs`?
`initframs` es iun archivo comprimido que contiene un pequeño sistema de archivos usado temporalmente durante el arranque del sistema y se almacena en `/boot/initrd-img-<version>`.
Se genera o actualiza:
- al instalar o actualizar el kernel
- al instalar nuevos módulos (necesarios para el arranque) del kernel
- al modificar configuraciones relacionadas al arranque 

10. ¿Qué módulos y drivers deberá tener un `initramfs` mínimamente para cumplir su objetivo?
Como el rol del `initframs` es peritir que es preparar lo mínimo necesario para que el sistema pueda montar y pasarle el control al sistema operativo entonces mínimamente el arranque del sistema deberá incluir los siguientes módulos: 
- drivers de almacenamiento: permitir acceder al disco físico donde está el sistema. 
- drivers de sistema de archivos: permiten montar el sistema de archivos raíz (`/`) como ext4, btrfs, vfat. 
- módulos del bus del sistema: para reconocer el hardware al que están conectados los discos. 
![alt text](image-9.png) _Módulos actuales del sistema_ 

### Práctica guiada 

#### Desarrollo de un módulo simple
1. Crear el archivo memory.c con el siguiente contenido:
```c
#include <linux/module.h>

MODULE_LICENSE("Dual BSD/GPL");
``` 
2. Crear el archivo Makefile con el siguiente contenido:
```bash
obj-m := memory.o //
``` 
Responder: 
- Explique brevemente cual es la utilidad del archivo `Makefile`.
    Sirve para compilar el módulo.  
- ¿Para qué sirve la macro MODULE_LICENSE? ¿Es obligatoria? 
    Se usa para indicar la licencia bajo la cual se distribuye el módulo. No es obligatoria pero es recomendable incluirla para evitar problemas legales y de compatibilidad con el kernel.

3. Compilar el módulo usando el mismo kernel en que correrá el mismo, utilizaremos el que instalaos en el primer paso del ejercicio guiado. 
```bash
make -C <KERNEL_CODE> M=$(pwd) modules //En KERNEL_CODE se debe poner la ruta del kernel que se descargo.
``` 
a. ¿Cuál es la salida del comando anterior?
```bash
root@so:/home/so/practica2# make -C /home/so/kernel/linux-6.13  M=$(pwd) modules
make: se entra en el directorio '/home/so/kernel/linux-6.13'
make[1]: se entra en el directorio '/home/so/practica2'
  CC [M]  memory.o
  MODPOST Module.symvers
  CC [M]  memory.mod.o
  CC [M]  .module-common.o
  LD [M]  memory.ko
make[1]: se sale del directorio '/home/so/practica2'
make: se sale del directorio '/home/so/kernel/linux-6.13'
```

b. ¿Qué tipos de archivo se generan? Explique para qué sirve cada uno.
Se generaron los siguientes archivos: 
- `memory.o`: es el objeto del módulo compilado. Contiene el código máquina del módulo.
- `memory.mod.o`: contiene información adicional sobre el módulo, como su nombre, versión y dependencias.
- `memory.ko`: es el módulo del kernel compilado (kernel object). Es el archivo que se carga en el kernel para usar la funcionalidad del módulo.

c. Con lo visto en la Práctica 1 sobre Makefiles, construya un Makefile de manera que
si ejecuto
    i. `make`, nuestro módulo se compila
    ii. `make clean`, limpia el módulo y el código objeto generado
    iii. `make run`, ejecuta el programa


4. El paso que resta es agregar y eventualmente quitar nuestro módulo al kernel en tiempo de ejecución.
Ejecutamos:
```bash	
insmod memory.ko
```
Responda lo siguiente: ¿Para qué sirven el comando insmod y el comando modprobe? ¿En qué se diferencian?
    Tanto `insmod` como `modprobe` se utilizan para cargar módulos en el kernel.
    - `insmod`: carga un módulo específico en el kernel. No resuelve dependencias automáticamente, por lo que si el módulo requiere otros módulos, deben ser cargados manualmente primero.
    - `modprobe`: carga un módulo y resuelve automáticamente las dependencias necesarias. Si el módulo requiere otros módulos, `modprobe` los carga en el orden correcto. Es más conveniente para manejar módulos con dependencias.

5. Verificamos que el módulo se haya cargado correctamente:
```bash
lsmod | grep memory
```
Salida: 
```bash
root@so:/home/so/practica2# lsmod | grep memory
memory                  8192  0
```
a. ¿Cuál es la salida del comando? Explique cuál es la utilidad del comando lsmod.
Indica que el módulo `memory` se ha cargado correctamente en el kernel.
`lsmod` muestra una lista de todos los módulos del kernel que están actualmente cargados en el sistema. Proporciona información sobre el nombre del módulo, su tamaño y cuántas veces está siendo utilizado (número de referencias).

b. ¿Qué información encuentra en el archivo `/proc/modules`?
Es un archivo virtual del sistema, que contiene información en tiempo real sobre los módulos cargados, es la fuente directa de donde `lsmod` obtiene la información y la formatea.

c. Si ejecutamos `more /proc/modules` encontramos los siguientes fragmentos ¿Qué información obtenemos de aquí?:
```bash
memory 8192 0 - Live 0x0000000000000000 (OE)
binfmt_misc 24576 1 - Live 0x0000000000000000
intel_rapl_msr 16384 0 - Live 0x0000000000000000
intel_rapl_common 32768 1 intel_rapl_msr, Live 0x0000000000000000

```
De los fragmentos se puede ver el nombre del módulo, su tamaño, el número de referencias (0 significa que no hay otros módulos que dependan de él), y el estado del módulo (Live indica que está activo). También se puede ver la dirección de memoria donde está cargado el módulo.

d. ¿Con qué comando descargamos el módulo de la memoria?
```bash
rmmod memory
```

6. Descargue el módulo memory. Para corroborar ejecute: 
```bash
lsmod | grep memory
```
7. Modifique el archivo memory.c:
```c	
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h> 
MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void) {
    printk(KERN_INFO "Hello, world!\n");
    return 0;
}

static void hello_exit(void) {
    printk("Bye, cruel world!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```
- Cargar el modulo, ejecutar `dmesg`, descargar el modulo y ejecutar `dmesg` nuevamente. ¿Qué diferencias encuentra?
![alt text](image-10.png)

8. Responder:
a. ¿Para qué sirve la función `module_init`? ¿Y `module_exit`? ¿Cómo haría para ver la información del log que arrojan las mismas?
`module_init` se usa para indicar la función que se ejecutará cuando el módulo se cargue en el kernel. `module_exit` indica la función que se ejecutará cuando el módulo se descargue del kernel. 
Para ver la información del log que arrojan se puede usar `dmesg` o `cat /var/log/kern.log`. 

b. Hasta aquí hemos desarrollado, compilado, cargado y descargado un módulo en nuestro kernel. En este punto y sin mirar lo que sigue. ¿Qué nos falta para tener un driver completo?.
Para tener un driver completo faltaría:
- Implementar la lógica específica del hardware que se va a controlar.
- Manejar la comunicación entre el kernel y el hardware (lectura/escritura de datos).

c. Clasifique los tipos de dispositivos en Linux. Explique las características de cada uno.
Se clasifican en:
- **Dispositivos de bloque**: permiten acceso aleatorio a bloques de datos. Ej: discos duros, pendrives. Se accede a ellos mediante bloques de tamaño fijo.
- **Dispositivos de carácter**: permiten acceso secuencial a datos. Ej: teclados, mouse. Se accede a ellos byte por byte y 1 byte solo puede ser leído por unica vez.

#### Desarrollo de un driver
Ahora completamos nuestro módulo para agregarle la capacidad de escribir y leer un
dispositivo. En nuestro caso el dispositivo a leer será la memoria de nuestra CPU, pero podría ser
cualquier otro dispositivo.

> El major number identifica qué driver del kernel maneja un determinado dispositivo. Cuando se accede a un archivo en /dev, el kernel usa el número mayor para saber a qué driver llamar.

1. Modifique el archivo memory.c para que tenga el siguiente código:
https://gitlab.com/unlp-so/codigo-para-practicas/-/blob/main/practica2/crear_driver/1_memory.c

2. Responder:
a. ¿Para qué sirve la estructura ssize_t y memory_fops? ¿Y las funciones
register_chrdev y unregister_chrdev?
`ssize_t` es un tipo de dato entero con signo. Se utiliza para representar el tamaño de un objeto en bytes leídos o escritos exitosamente (valor positivo) o un código de error (valor negativo). En el código se usa: si la función `memory_read()` devuelve 1, significa que leyó 1 byte, en cambio si devuelve -EFAULT, hubo un error de acceso a memoria del usuario.
`memory_fops` es una estructura del kernel que define las funciones que implementa el driver para operaciones comunes de archivos. En el caso del código se definen las operaciones y se le pasan las funciones que se implementaron. 

Las funciones `register_chrdev` y `unregister_chrdev` se utilizan para registrar y anular el registro de un controlador de caracteres (character device driver) en el kernel. 
- `register_chrdev`: registra un dispositivo de caracteres con un numero mayor. Le da un nombre y las funciones que se usarán para manejar las operaciones de lectura y escritura.
- `unregister_chrdev`: anula el registro del dispositivo de caracteres. Limpio todo cuando el módulo se remueve (rmmod).

b. ¿Cómo sabe el kernel que funciones del driver invocar para leer y escribir al
dispositivo?
Lo sabe por la estructura `file_operations` que se le pasa a `register_chrdev`. Esta estructura contiene punteros a las funciones que implementan las operaciones. 

c. ¿Cómo se accede desde el espacio de usuario a los dispositivos en Linux?
Los dispositivos se acceden a través de archivos especiales ubicados en `/dev`. Estos archivos representan los dispositivos y permiten interactuar con ellos como si fueran archivos normales. Por ejemplo, desde el espacio de usuario se puede usar `open()` para abrir un archivo de dispositivo y su equivalente en bash sería `cat`, `echo`, etc.  

d. ¿Cómo se asocia el módulo que implementa nuestro driver con el dispositivo?
Se da en `register_chrdev`. Se le pasa el número mayor y el nombre del dispositivo. El kernel asocia el número mayor con el módulo que implementa el driver. Cuando se accede al dispositivo, el kernel usa este número para invocar las funciones definidas en `memory_fops`. Luego se crean un archivo en `/dev` con el nombre del dispositivo. 
```bash
mknod /dev/memory c <major_number> 0
```
Con `unregister_chrdev` se desasocia el módulo del dispositivo.


e. ¿Qué hacen las funciones `copy_to_user` y `copy_from_user`?
(https://developer.ibm.com/articles/l-kernel-memory-access/) [^2].
- `copy_to_user`: se encarga de copiar datos desde el espacio del kernel al espacio de usuario. 
- `copy_from_user`: se encarga de copiar datos desde el espacio de usuario al espacio del kernel.
Ambas funciones son necesarias porque el kernel y el espacio de usuario tienen diferentes espacios de memoria y no pueden acceder directamente a los datos del otro. Estas funciones manejan la transferencia de datos entre ambos espacios de manera segura, verificando permisos y evitando accesos no autorizados.

3. Ejecutar:
```bash
mknod /dev/memory c 60 0
``` 
4. Ejecutar: 
```bash
insmod memory.ko
```
Responder: 
a. ¿Para qué sirve el comando `mknod`? ¿qué especifican cada uno de sus parámetros?
Sirve para crear archivos de dispositivo en `/dev`.
```bash
mknod [ruta] [tipo] [mayor] [menor]
```
- **ruta**: ruta donde se creará el archivo de dispositivo (ej: `/dev/memory`).
- **tipo**: tipo de dispositivo (c para carácter, b para bloque).
- **mayor**: número mayor del dispositivo (identifica el driver que maneja el dispositivo).
- **menor**: número menor del dispositivo (identifica una instancia específica del dispositivo manejado por el driver).

b. ¿Qué son el "major number" y el "minor number"?
El **major number** (número mayor) es un identificador único que asigna el kernel a un driver específico. Indica qué driver manejará el dispositivo.
El **minor number** (número menor) es un identificador que permite distinguir entre diferentes instancias de un mismo dispositivo manejado por el mismo driver. Por ejemplo, si hay dos discos duros manejados por el mismo driver, cada uno tendrá un número menor diferente.

5. Ejecutar:
```bash
echo -n abcdef > /dev/memory
``` 
6. Ejecutar:
```bash
more /dev/memory
```
![alt text](image-11.png) _Ejecutamos el Makefile, seguimos los pasos indicados. Se puede ver que al ser declados como dispositivo por caracter lo que queda en el buffer es el último char escrito. En caso de ser bloque, quedaría el bloque completo._

7. Responder:
a. ¿Qué salida tiene el anterior comando?, ¿Porque? (ayuda: siga la ejecución de las funciones memory_read y memory_write y verifique con dmesg)
```bash
[ 6086.806185] memory_write()
[ 6086.806195] memory_write()
[ 6086.806197] memory_write()
[ 6086.806200] memory_write()
[ 6086.806203] memory_write()
[ 6086.806205] memory_write()
[ 6096.584641] memory_read()
[ 6096.584918] memory_read()
```
La salida es `f` porque el driver está implementado para que solo se guarde el último byte escrito en el buffer. Notar que la función `memory_write` se invoca 6 veces, una por cada byte escrito. 

b. ¿Cuántas invocaciones a memory_write se realizaron?
Se realizaron 6. 
c. ¿Cuál es el efecto del comando anterior? ¿Por qué?
El efecto es que se escriben los bytes `abcdef` en el buffer del driver. Pero como el driver está implementado para guardar solo el último byte, al final solo queda `f` en el buffer.

d. Hasta aquí hemos desarrollado un ejemplo de un driver muy simple pero de manera completa, en nuestro caso hemos escrito y leído desde un dispositivo que en este caso es la propia memoria de nuestro equipo. En el caso de un driver que lee un dispositivo como puede ser un file system, un dispositivo usb, etc. ¿Qué otros aspectos deberíamos considerar que aquí hemos omitido? ayuda: semáforos, ioctl, inb, outb.

Se deben considerar aspectos como:
- Manejo de concurrencia: usar semáforos o mutex para evitar condiciones de carrera al acceder a recursos compartidos.
- Implementar funciones `ioctl` para manejar operaciones de entrada/salida específicas del dispositivo.
- Manejo de interrupciones: usar `inb` y `outb` para leer y escribir directamente en puertos de hardware.
- Manejo de errores: verificar errores en las operaciones de lectura/escritura y devolver códigos de error apropiados.


[^1]: https://www.ibm.com/docs/es/aix/7.3?topic=concepts-kernel-environment#kernextc_kern_env
[^2]: Modifiqué el link original. El contenido se encuentra archivado. Deprecado. 