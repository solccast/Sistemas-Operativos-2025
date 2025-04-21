# Sistemas Operativos - Práctica 3 [2025]

## Threading (ULT y KLT)

### Conceptos generales

1. ¿Cuál es la diferencia fundamental entre un proceso y un thread?
La diferencia clave es que en el caso del thread cuando se genera éste vive en el espacio de direcciones que el proceso que lo creó y compartirá los recursos, en cambio en el caso de crear un nuevo proceso (mediante fork) éste contará con un nuevo espacio de direcciones con su propio contexto. 

> Un hilo es la unidad de trabajo que se puede someter a ejecución. Se ejecuta secuencialmente y es interrumpible para que el procesador pase a otro hilo. 


El uso de hilos en reemplazo de los procesos posee ventajas:
| | **Procesos** | **Hilos** |
| -- | -- | -- |
| **Context Switch**    | El SO debe intervenir para guardar el ambiente del proceso saliente y recuperar el ambiente del nuevo | Se realiza a nivel de registros y no espacio de direcciones. Lo lleva a cabo el proceso   |
| **Creación**  | Creación de un nuevo espacio de direcciones (PCB, PC). Actua el Kernel    |  Creación de una TCB, registros, PC y un espacio para el stack. Lo hace el mismo proceso sin intervención del SO. |
| **Destrucción**   | El SO interviene para eliminar la PCB del proceso saliente.   | El proceso elmina la TCB. |
| **Planificación** | Cambio de contexto continuos llevados por el SO.  | A cargo del desarrollador.    |
| **Protección**    | El SO garantiza la seguridad entre procesos.  | Un hilo puede bloquear la ejecución de otros. Todos los hilos comparten el mismo espacio de direcciones.  |

>Un proceso puede verse como una colección de uno o más hilos. Para ejecutar otro proceso, se debe llevar adelante un cambio de contexto. 

2. ¿Qué son los User-Level Threads (ULT) y cómo se diferencian de los Kernel-Level Threads (KLT)?

- **ULT**: Son hilos gestionados completamente por una biblioteca a nivel de usuario sin intervención directa del kernel. **El kernel _no se entera_ de la existencia de Threads**.
    - **Ventajas**: 
        - Comparten el espacio de direcciones. 
        - Cada proceso las planifica como le conviene. 
        - Portables ya que no dependen de características específicas del sistema operativo. 
        - Más rápidos de crear y cambiar entre ellos. 
    - **Desventajas**:
        - No se pueden ejecutar hilos del mismo proceso en distintos procesadores. 
        - Si un hilo hace una llamada bloqueante al sistema, todo el proceso se bloquea.
        - Un hilo puede monopolizar el uso de la CPU por parte del proceso. 

- **KLT**: gestionados directamente por el kernel, entonces cada hilo es conocido por éste. 
    - **Ventajas**:
        -  Si un hilo se bloquea, el kernel puede ejecutar otro hilo del mismo proceso.
        - Se pueden ejecutar en paralelo en múltiples CPUs o núcleos.
    - **Desventajas**: 
        - Cambiar de hilo implica un cambio de contexto.
        - Mator sobreccarga para crear y gestionar hilos. 

> En muchos sistemas modernos, se usa una combinación de ambos a través del modelo "many-to-many", donde múltiples ULT pueden ser mapeados a múltiples KLT.

3. ¿Quién es responsable de la planificación de los ULT? ¿y los KLT? ¿Cómo afecta esto al rendimiento en sistemas con múltiples núcleos?
En el caso de los ULT el responsable es la biblioteca en espacio de usuario y en KLT se ocupa directamente el kernel que ve y programa cada hilo como una unidad separada de ejecución.
En sistemas con múltiples núcleos los KLT se aprovechan mucho más ya que el kernel puede distribuir los hilos entre núcleos. Los hilos de un mismo proceso pueden estar ejecutándose concurrentemente en los distintos procesadores

4. ¿Cómo maneja el sistema operativo los KLT y en qué se diferencian de los procesos?
En Linux, por ejemplo, los hilos se implementan como procesos especiales usando `clone()`, y cada hilo puede tener su propio ID de hilo (TID) pero compartir recursos como memoria, archivos, etc., con sus "hermanos" del mismo proceso.

> La diferencia clave con los procesos es que los hilos comparten el mismo espacio de memoria y recursos, mientras que los procesos son más independientes y aislados.

5. ¿Qué ventajas tienen los KLT sobre los ULT? ¿Cuáles son sus desventajas?
Este tema fue desarrollado más arriba. Cuadro resumen:


6. Qué retornan las siguientes funciones:
a. `getpid()` [^1]: devuelve el ID de proceso del proceso de llamada. 
b. `getppid()` [^1]: devuelve el ID de proceso del proceso padre del proceso de llamada. 
c. `gettid()`[^2]: obtiene el identificador de hilo. En un miltiproceso todos los subprocesos tienen el mismo PID pero cada uno tiene un TID único. 
d. `pthread_self()`: devuelve el ID del hilo de llamada. 
e. `pth_self()` [^3]: devuelve el ID único del hilo en ejecución. 

7. ¿Qué mecanismos de sincronización se pueden usar? ¿Es necesario usar mecanismos de sincronización si se usan ULT?
Se pueden usar: 
- Mutex: un hilo accede a un recurso a la vez.
- Semáforos
- Barreras
- Conidiciones

Es necesario sincronizar tanto en KLT como en ULT si es que acceden a recursos compartidos. 

8. **Procesos**
    a. ¿Qué utilidad tiene ejecutar `fork()` sin ejecutar `exec()`?
    Primero tener en cuenta que al ejecutar  `fork()` crea un nuevo proceso hijo que es una copia exacta del proceso padre (ejecuta desde la línea siguiente al `fork()`). Sirve para hacer trabajo paralelo dentro del mismo programa, es útil si el hijo va a seguir ejecutando el mismo programa pero con distinto comportamiento. 
    b. ¿Qué utilidad tiene ejecutar `fork()` + `exec()`?
    Lanza un programa nuevo desde otro. `fork()` crea el proceso hijo y `exec()` en el hijo reemplaza su código y memoria por el de otro programa desde uno ya en ejecución (como una shell). El proceso padre puede continuar ejecutándose o esperar al hijo.  
    c. ¿Cuál de las 2 asigna un nuevo PID `fork()` o `exec()`?
    La instrucción `fork()` es la que se encarga de crear un nuevo proceso con un nuevo PID, en cambio `exec()` lo mantiene pero reemplaza el contenido del proceso actual. 
    d. ¿Qué implica el uso de Copy-On-Write (COW) cuando se hace `fork()`?
    Cuando un proceso crea un proceso hijo inicialmente compartirán las mismas páginas en memoria y éstas serán marcados como COW lo que significa que si alguno de estos procesos trata de modificarlas entonces se creará una copia de la página y el proceso trabajará sobre ella para que no afecte al resto de procesos[^4]. 
    ![alt text](image.png)
    e. ¿Qué consecuencias tiene no hacer `wait()` sobre un proceso hijo?
    En caso de que eso ocurra el proceso hijo queda como un proceso zombi: el proceso hijo terminó pero su entrada en la tabla de procesos sigue existiendo para que el padre pueda leer su estado de salida por lo que ocupa recursos del sistema. 
    f. ¿Quién tendrá la responsabilidad de hacer el `wait()` si el proceso padre termina sin hacer `wait()`?
    En este caso el que tiene la responsabilidad es tomado por el proceso `init` o `systemd` (PID 1)

9. **Kernel Level Threads**
    a. ¿Qué elementos del espacio de direcciones comparten los threads creados con `pthread_create()`?
    Al ser creados por el mismo proceso comparten: segmento de código, segmento de datos, heap, etc. Entonces: comparten todo el espacio de direcciones del proceso, salvo sus propias pilas y registros.  
    b. ¿Qué relaciones hay entre `getpid()` y `gettid()` en los KLT?
    Todos los hilos de un proceso tienen el mismo PID, pero cada uno tiene su propio TID. 
    c. ¿Por qué `pthread_join()` es importante en programas que usan múltiples hilos? ¿Cuándo se liberan los recursos de un hilo zombie?
    Es importante ya que espera a que el hilo termine y recupera su valor de retorno y evita que quede como zombie. Los recursos asociados no se liberan hasta que se hace `pthread_join`.  
    d. ¿Qué pasaría si un hilo del proceso bloquea en `read()`? ¿Afecta a los demás hilos?
    No, el kernel solo bloquea ese hilo pero no a todo el proceso. 
    e. Describí qué ocurre a nivel de sistema operativo cuando se invoca `pthread_create()` (¿es syscall? ¿usa `clone`?).
    Al ejecutar `pthread_create()` se hace una llamada al sistema (`clone()`) que permite crear un nuevo hilo. Esta syscall permite especificar qué recursos se comparten. El kernel crea una nueva estructura de hilo (`task_struct`). 

10. **User Level Threads**
a. ¿Por qué los ULTs no se pueden ejecutar en paralelo sobre múltiples núcleos?
No se pueden ejecutar porque en paralelo porque el SO desconoce la existencia de estos hilos por lo tanto solo uno puede ejecutarse a la vez. 


b. ¿Qué ventajas tiene el uso de ULTs respecto de los KLTs?
        Se hizo enfásis en las ventajas en puntos anteriores, sin embargo como resumen tenemos que ULTs tiene como ventaja por sobre KLTs:
            - Menor overhead
            - Portabilidad
            - Mayor control de planificación
            - Cambio de contexto más rápido

c. ¿Qué relaciones hay entre `getpid()`, `gettid()` y `pth_self()` (en GNU Pth)?

d. ¿Qué pasaría si un ULT realiza una syscall bloqueante como `read()`?
        Todo el proceso se bloquea. Como mencioné antes, el kernel desconoce la existencia de estos hilos entonces ve a todos estos como un proceso por lo que el resto de ULTs se quedan esperando. 
e. ¿Qué tipos de scheduling pueden tener los ULTs? ¿Cuál es el más común?
    Los más comunes:
        - Cooperativo: los hilos ceden voluntariamente el control.
        - Round Robin: turno fijo acada hilo en orden.
        - Por prioridades

> No hay paralelismo real, solo concurrencia cooperativa dentro de un solo núcleo. 

11. **Global Interpreter Lock**
a. ¿Qué es el GIL (Global Interpreter Lock)? ¿Qué impacto tiene sobre programas multi-thread en Python y Ruby?
El GIL se refiere al mecanismo interno de control que garantiza que solo un hilo de ejecución acceda a objetos en un momento dado. Previene las _race conditions_. En conclusión, permite que **solo un hijo ejecute bytecode a la vez** en el intérprete. [^5]

b. ¿Por qué en CPython o MRI se recomienda usar procesos en vez de hilos para tareas intensivas en CPU?
Porque los hilos están limitados por el GIL así que no pueden correr en paralelo real, en cambio cada proceso tiene su propio intérprete y su propio GIL por lo que pueden ejecutarse en simultáneo. 

### Práctica guiada 


[^1]: https://www.ibm.com/docs/es/aix/7.2?topic=g-getpid-getpgrp-getppid-subroutine 
[^2]: https://www.ibm.com/docs/es/aix/7.2?topic=p-pthread-self-subroutine
[^3]: https://www.gnu.org/software/pth/pth-manual.html
[^4]: https://www.geeksforgeeks.org/copy-on-write/
[^5]: https://wiki.python.org/moin/GlobalInterpreterLock