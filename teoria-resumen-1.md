# Sistemas Operativos - Teoría - 1

1. ¿Cuál es la función principal de un sistema operativo desde la perspectiva del usuario? ¿Y desde la perspectiva del sistema?
- Desde la perspectiva del usuario, un SO proporciona una interfaz cómoda y amigable, ocultando la complejidad del hardware. Desde la perspectiva del sistema, el SO administra los recursos del hardware y provee servicios a los procesos.
2. Menciona tres objetivos fundamentales de los sistemas operativos. Explica brevemente uno de ellos.
- Los tres objetivos son: 
    - comodidad (facilitar el uso del hardware), 
    - eficiencia (uso óptimo de los recursos) y 
    - evolución (permitir nuevas funcionalidades). 
    La eficiencia, por ejemplo, busca maximizar el rendimiento del sistema utilizando la CPU, memoria y dispositivos de E/S de la manera más efectiva posible.
3. ¿Qué es el kernel y cuál es su rol principal en un sistema operativo? ¿Dónde se encuentra ubicado generalmente?
- El kernel es la porción central del sistema operativo que reside en la memoria principal. Su rol principal es administrar los recursos del sistema, incluyendo la CPU, la memoria, los dispositivos de entrada/salida y los procesos.
4. Describe el propósito de las "llamadas al sistema" (system calls). ¿Quiénes las utilizan y para qué?
- Las llamadas al sistema son la interfaz que los procesos de usuario utilizan para solicitar servicios al kernel que requieren acceso al hardware o a funcionalidades privilegiadas del sistema operativo. Permiten a los programas interactuar con el SO para realizar tareas como leer archivos o crear nuevos procesos.
5. Explica la diferencia fundamental entre el modo de ejecución supervisor (kernel) y el modo de ejecución usuario. ¿Por qué es necesaria esta distinción?
- En modo supervisor (kernel), el código tiene acceso completo a todas las instrucciones de la CPU y a la totalidad de la memoria y el hardware. En modo usuario, el acceso está restringido a un espacio de direcciones propio del proceso. Esta distinción es necesaria para proteger la integridad del sistema operativo y evitar que procesos de usuario malintencionados o erróneos afecten a otros procesos o al propio kernel.
6. ¿Cómo se produce la transición del modo usuario al modo kernel? ¿Quién o qué mecanismo inicia este cambio?
- La transición del modo usuario al modo kernel ocurre a través de una interrupción (externa al proceso, como una señal de un dispositivo) o un trap (generado por el propio proceso, como una llamada al sistema o una excepción). El hardware, en respuesta a estas, cambia el bit de modo a kernel.
7. Describe las características principales de un kernel monolítico. Menciona una ventaja y una desventaja de este tipo de arquitectura.
- Un kernel monolítico incluye todos los servicios del sistema operativo (gestión de procesos, memoria, archivos, controladores, etc.) en un único bloque de código que se ejecuta en modo supervisor. Una ventaja es su potencial eficiencia al no requerir cambios de modo frecuentes. Una desventaja es su complejidad, lo que dificulta su mantenimiento y puede aumentar el riesgo de fallos en todo el sistema.
8. ¿Cuáles son las principales ventajas de un microkernel en comparación con un kernel monolítico? ¿Cuál suele ser su principal inconveniente?
9. ¿Qué diferencia a un kernel híbrido de un kernel puramente monolítico o un microkernel? Menciona un ejemplo de un sistema operativo con kernel híbrido.
- Las principales ventajas de un microkernel son su menor tamaño, mayor modularidad (lo que facilita la personalización), mayor estabilidad (los fallos en los servicios de usuario no afectan al kernel) y mayor seguridad (al ejecutar la mayoría de los servicios en modo usuario). Su principal inconveniente suele ser un rendimiento inferior debido a la necesidad de frecuentes cambios de modo para la comunicación entre procesos y servicios.
- Un kernel híbrido combina características de los kernels monolíticos y microkernels, teniendo un núcleo relativamente pequeño que ejecuta algunos servicios esenciales en modo kernel para mejorar la eficiencia, mientras que otros servicios se ejecutan en modo usuario para aumentar la seguridad y modularidad. Windows y MacOS son ejemplos de sistemas operativos con kernels híbridos.
10. ¿Cuál es el requisito fundamental de un kernel de tiempo real? Menciona una técnica o característica que utilizan para cumplir con este requisito.
- El requisito fundamental de un kernel de tiempo real es garantizar que las operaciones se completen dentro de marcos de tiempo conocidos y predecibles. Utilizan algoritmos de scheduling de CPU avanzados que priorizan tareas críticas y mecanismos de asignación de memoria optimizados para evitar retrasos, además de soporte para multitasking apropiativo.

