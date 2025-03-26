# Sistemas Operativos - Teoría - 1

## System Call: llamadas al sistema
Una llamada al sistema se define como **el mecanismo utilizado por un _proceso de usuario_ para solicitar un servicio al Sistema Operativo (SO)**.
Un proceso de usuario ejecuta código **propio** en un entorno limitado y no tiene acceso directo al hardware, ientras que el SO se ejecuta en modo Kernel y es responsable de gestionar los procesos de forma controlada. 
> Proceso de usuario -> opera -> entorno controlado y limitado de ejecución. Restringido a su propio espacio de direcciones. 
Las llamadas al sistema son el método a través del cual un proceso de usuario puede invocar servicios protegidos del SO. El SO provee una interfaz de programación (API) que los procesos pueden usar para solicitar recursos gestionados por él. El SO actúa como un servidor que recibe estas solicitudes y determina la mejor manera de satisfacerlas, definiendo los parámetros que el programador debe enviar y los valores de retorno. En Unix y GNU/Linux, esta API se llama `libc`. 

## Librería `libc`
`libc` es la API principal l del SO, provee las librerías estándar de C y actúa como una interfaz entre aplicaciones de usuario y las System Calls (System Call Wrappers). Un ejemplo de llamada al sistema es read(file, buffer, nbytes), utilizada para leer un archivo.

### Ejemplo de system call: read
`read(file, buffer, nbytes)` es una syscall utilizada para leer un archivo. Se siguen los siguientes pasos: 
• Primero, se colocan los parámetros necesarios (archivo, buffer, número de bytes) en la pila.
• Luego, se invoca la función read implementada en la librería de usuario y comienza su ejecución. Esta función es sencilla y se encarga de indicar el número de la llamada al sistema y realizar la invocación.
• La función read ejecuta una TRAP (Interrupción por Software) para cambiar del Modo Usuario al Modo Kernel y pasar el control al SO. Para todas las llamadas al sistema se utiliza la misma interrupción.
• La llamada al sistema invocada se identifica a través del valor de un registro.
•Una vez que el SO tiene el control, verifica cuál es la llamada al sistema que debe atender y ejecuta el código correspondiente, como acceder al dispositivo de almacenamiento para leer el archivo solicitado en memoria en el caso de read.

> El código de la librería (como libc) se encarga de indicar el número de syscall a ejecutar, los parámetros y luego emite un aviso al SO (Trap) para gestionar la syscall.

En GNU/Linux, las system calls se identifican por un número y pueden tener un máximo de seis parámetros. Las tablas que definen estas llamadas para x86 en 32 y 64 bits se encuentran en `arch/x86/entry/syscalls/syscall_32.tbl` y `arch/x86/entry/syscalls/syscall_64.tbl` respectivamente. El dispatcher, al producirse una interrupción, verifica el número en la tabla y ejecuta la función asociada.

## Registros en Arquitectura x86 (32 bits):
Se mencionan los registros utilizados para pasar el número de la syscall y sus parámetros en la arquitectura x86 de 32 bits:
- "EAX lleva el numero de syscall que se desea ejecutar".
- "EBX lleva el primer parámetro".
- "ECX lleva el segundo parámetro".
Y así sucesivamente para EDX, ESI y EDI.
La instrucción que inicia la system call en 32 bits es "int 80h" (para 64 bits es diferente).

Es crucial manejar los parámetros de las system calls con cuidado, ya que provienen del espacio de usuario y no se puede asumir que sean correctos. Los punteros no pueden apuntar al espacio del Kernel por seguridad y deben ser siempre válidos para evitar un **Kernel Panic** (en un ejemplo como read, un buffer podría tener una dirección del Kernel y sobrescribir datos sensibles). El Kernel utiliza APIs especiales (`get_user()`, `put_user()`, `copy_from_user()`, `copy_to_user()`) para acceder de forma segura al espacio de direcciones del proceso que invocó la system call.


> En UNIX, `libc` es la API principal para la invocación de las System Calls. Provee las librerías estándar de C y actúa como una interfaz (System Call Wrappers) entre las aplicaciones de usuario y las System Calls. La funcionalidad de libc y las System Calls está definida por el estándar POSIX, que busca proveer una interfaz común para lograr portabilidad, de manera que la interacción sea con la API y no directamente con el Kernel.