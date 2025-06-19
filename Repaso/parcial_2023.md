1. ¿Qué es el kernel de Linux? ¿Cuáles son sus funciones principales dentro del SO?
El kernel de linux es el núcleo del sistema operativo, la capa de software que se encarga de la comunicación entre el hardware y las aplicaciones. Es del tipo monolitico híbrido que indica que contiene muchas funcionaldiades integradas pero permite la carga dinámica de módulos. 
Funciones:  
- Gestión de procesos: manejo de prioridades entre procesos, planificación (scheduling), asignación de tiempo de CPU a los procesos. 
- Gestión de memoria: asignación y liberación de memoria, administra la memoria RAM. 
- Gestión de dispositivos: comunicación con el hardware mediante drivers. 
- Gestión de filesystem 
- Gestión de seguridad y permisos: proporciona aislamiento entre procesos para evitar acceso no autorizados. 

2. Explique brevemente qué es un módulo del kernel de Linux y qué ventajas provee compilar una funcionalidad como módulo respecto a compilarla como built-in.
Un módulo del kernel es un bloque de código que adiciona funcionalidad al kernel de Linux dado que la arquitectura del kernel de Linux es monolito híbrido significa que se puede cargar/descargar módulos de manera dinámica bajo demanda. Built-in implica que el módulo se carga al kernel, o sea, al iniciar el sistema se cargará directamente monolito sea menos liviano y el kernel se tenga que recompilar completo, en cambio compilar funcionalidad como módulo se puede realizar en tiempo de ejecución tanto cargarla como descargarla.  

3. ¿Qué tipos de archivos se encuentran en `/dev`? ¿Qué representan estos archivos?
En la carpeta `/dev/` se encuentran archivos de los dispositivos que representan a hardware o recursos manejados por drivers. Pueden ser archivos de dispositivos de bloque (como pendrives, discos) o de caracter (teclados, mouse).
4. Verdadero/Falso
    - libc es el componente del kernel donde se definen las system calls. -> Falso. Libc es la librería de epsacio de usuario que actúa como wrapper para las llamads al sistema dele kernel.  
    - las system calls se definen en módulos del kernel que se pueden administrar con insmod, modprobe, rmmod y modinfo. -> Falso. Las instrucciones mencionadas son para la administración de módulos en tiempo de ejecución. 
    - las syscalls se ejecutan en modo privilegiado y se identifican a través de un número -> Verdadero. 
    - Cada driver implementa una system call. -> falso. Cada driver implementa funcionalidad asociada a un dispositivo.  
7. Suponga que un usuario desea limitar el uso de CPU de un proceso para que no sea mayor al 80%, ¿qué mecanismo provisto por el kernel Linux podría usar?
Podría usar cgroups ya que este se encarga de limitar los recursos que puede usar un proceso. Por ejemplo, en el caso de usar Cgroup V2 se puede definir un controlador para el recurso CPU en el archivo cpu.max = ...
8. En el contexto de Docker, ¿qué es una imagen? ¿y un contenedor? ¿cual es la principal diferencia entre ambos?
Una imagen es un conjunto de capas no editables (de solo lectura) que se usa como base para generar los contenedores. Lo que lo diferencia de un contenedor es que es éste es una instancia de una imagen y además posee una capa superior extra que es de lectura y escritura. Con una sola imagen se pueden levantar múltiples instancias y que cada uno tenga su propia capa de escritura diferente. 
9. ¿Qué características del kernel linux utiliza docker para proveer containers? 
Docker usa: 
- chroot: para que el código de un contenedor quede aislado del entorno
- namespaces: que el contenedor tenga una instancia de los recursos de forma aislada
- unión file systems: para generar los puntos de montaje de las capas 
- cgroups: restringir el uso de los recursos y un contenedor no acapare todo el sistema. 

10. ¿Qué es el archivo docker-compose y cuál es su función? ¿En qué sentido es diferente de un Dockerfile? 
Un archivo compose sirve para poder levantar un programa, aplicación que requiere de múltiples servicios. En él se pueden definir reglas sobre la conexión (red) que hay entre los servicios, las variables de entorno, reglas de reinicio, mapeo de puertos locales al contenedor con el del host, a partir de qué imagen construir y ejecutar, etc. En cambio un dockerfile es un archivo que indica cómo se construye una imagen en especial. A partir de un dockerfile obtenemos una imagen, a partir de docker compose obtenemos contenedores. 