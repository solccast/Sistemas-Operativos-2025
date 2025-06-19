1. Define qué es un container, ¿en qué se diferencia de una VM? 
Un contenedor es una instancia de imagen en ejecución que se encuentra aislado del resto del sistema. Está compuestas por capas resultado de la unión del filesystem (imagen) que son de solo lectura y una última capa escribible. Una VM se encarga de virtualizar sistemas operativos sobre una arquitectura sin embargo, un contenedor se encarga de virtualizar a nivel proceso sobre un sistema operativo compartiendo el kernel del sistema anfitrión lo que lo hace mucho más liviano.  

4. Indique los motivos por lo cuales recompilaría un kernel.
- Agregar o quitar soporte a hardware específico.
- Aplicar parches de seguridad o correcciones de bugs. 
- Optimizar o personalizar el kernel para una arquitectura o necesidad específica. 
- Incluir módulos o funcionalidades nuevas del kernel. 

5. ¿Qué hace la system call fork() y qué valores retorna?
La system call fork() crea un nuevo proceso duplicando el proceso actual. 
La system call fork() genera una nueva instancia del programa con un nuevo PID a partir de la instrucción siguiente a donde se llama. Posee el mismo contexto que el padre. 
En el proceso padre, fork retorna el PID del proceso hijo, en el hijo retorna 0. 

6. ¿Qué contenido tienen los archivos .patch como los que se usaron en la práctica de compilación del kernel? ¿Qué ventaja tiene su uso respecto a bajar una versión nueva del kernel?
Es un archivo encargado de tener las diferencias entre una versión del kernel y otro, es a partir de un kernel base. Es mucho más liviano que bajar una versión nueva ya que la compilación sería completa en cambio, usando un archivo .patch se compilan solo las modificaciones y permiten mantener una versión base del kerne y aplicar solo los cambios necesarios.  
8. Si la función de inicialización de un módulo del kernel contiene la siguiente línea: `result = register_chrdev(memory_major, "memory", &memory_fops)` ¿qué podría decir sobre el módulo? ¿Qué tipo de funcionalidad implementa? 
Se puede decir que se está registrando un driver para un nuevo dispositivo (por el uso de memory_major esto significa que se agrega un nuevo número registrado para un dispositivo nuevo) del tipo byte a byte (a caracteres, como lo puede ser un teclado) llamado _memory_. l comportamiento del dispositivo se define mediante la estructura `memory_fops` 
9. ¿Qué hay en el directorio /dev? ¿Qué tipo de funcionalidad implementa? 
En el directorio /dev se encuentran los dispositivos representados como archivos que funciona como interfaz de comunicación entre el SO y el HW. 
10. ¿Qué es el directorio /proc? ¿Dónde se almacenan sus archivos?
En el directorio /proc encontramos archivos en los que se encuentran información sobre el sistema y los procesos que se están ejecutando en el sistema. No están almacenados en disco sino que son generados por el kernel en tiempo real. Podemos ver información sobre métricas, sobre el namespace que está usando, sobre el usuario y grupo que lo está ejecutando. Cada carpeta del proc está definido por /proc/<PID>
