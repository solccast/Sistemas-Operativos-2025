# Sistemas Operativos - Práctica 4B [2025]: Docker y Docker Compose 

## Docker 
1. Utilizando sus palabras, describa qué es Docker y enumere al menos dos beneficios que encuentre para el concepto de contenedores.

Docker es la teconología de organización (crear, desplegar, ejecutar, actualizar y administrar) en contenedores. Se usan estos como máquinas virtuales muy livianas y modulares. Los **contenedores** son unidades de software ejecutable que empaquetan el código de la aplicación con sus librerías y dependencias lo que permite que puedan ejecutarse en cualquier entorno [^3]. Utiliza los cgroups y los namespaces para dividir los procesos y ejecutarlos de manera independiente. El propósito de los contenedores es ejecutar varios procesos y aplicaciones por separado para que se pueda aprovechar mejor la infraestructura y, al mismo tiempo, conservar la seguridad que se obtendría con los sistemas individuales. [^1]
Los contenedores permiten que varios componentes de la aplicación compartan los recursos de una única instancia del sistema operativo anfitrión. Este uso compartido es similar a la forma en que un hipervisor permite que varias máquinas virtuales (VM) compartan la unidad central de procesamiento (CPU), la memoria y otros recursos de un único servidor de hardware[^2].

Ventajas: 
- **Restauración**: todas las imágenes cuentan con capas por lo tanto se la puede restaurar a una versión anterior. Respalda el engoque de desarrollo ágil.
- **Implementación rápida**: Cada proceso se encuentra en un contenedor distinto, por lo que puede compartirlos con aplicaciones nuevas rápidamente.

2. ¿Qué es una imagen? ¿y un contenedor? ¿ cuál es la principal diferencia entre ambos? 
Como se mencionó antes, un **contenedor** es una unidad de software ejecutable en el que se encuentra empaquetado el código y sus dependencias. 
Las **imágenes** [^4] contienen código fuente de la aplicación ejecutable y todas las herramientas, bibliotecas y dependencias que el código de la aplicación necesita para ejecutar como contenedor. Se componen de capas, y cada capa corresponde a una versión de la imagen. Cada vez que un desarrollador realiza cambios en la imagen, se crea una nueva capa superior y esta capa superior reemplaza a la capa superior anterior como la versión actual de la imagen. Las capas anteriores se guardan para retrocesos o para reutilizarlas en otros proyectos.

> Un contenedor es una instancia en ejecución de una imagen. Es el entorno aislado donde realmente se ejecuta la aplicación. A partir de la imagen, al crear el contenedor se crea otra capa (_capa de contenedor_) que es R/W pero no persiste más allá de la vida del contenedor. 

3. ¿Qué es **Union Filesystem**? ¿Cómo lo utiliza Docker? 
**Union Filesystem (UnionFS)** es un sistema de archivos que permite combinar múltiples capas (layers) de sistemas de archivos en una sola vista unificada. Es decir, puede montar varias capas como si fueran una sola, permitiendo que algunas sean de solo lectura y otras de lectura-escritura.
Docker usa UnionFS para construir y gestionar imágenes de forma eficiente. Las imágenes de Docker están compuestas por capas apiladas, donde cada capa representa un cambio (como instalar un paquete o copiar un archivo).
**Ventajas de usar UnionFS en Docker:**
- **Reutilización**: Si varias imágenes comparten una misma capa (como la base python:3.11), Docker la descarga una sola vez.
- **Eficiencia**: Solo se almacenan y transfieren los cambios.
- **Rapidez**: Al construir imágenes, Docker usa caché de capas anteriores si no han cambiado.
- **Inmutabilidad**: Las capas inferiores no se modifican, mejorando la seguridad y estabilidad.

4. ¿Qué rango de direcciones IP utilizan los contenedores cuando se crean? ¿De dónde la obtiene?
Por defecto, Docker asigna direcciones IP a los contenedores desde una subred interna cuando se utiliza la red puente (_bridge_). La red puente utiliza IPAM (IP Address Management) para asignar direcciones IP automáticamente lo que permite la comunicación con otros contenedores en el mismo puente[^5].
El rango más común es `172.17.0.0/16`

5. ¿De qué manera puede lograrse que las datos sean persistentes en Docker? ¿Qué dos maneras hay de hacerlo? ¿Cuáles son las diferencias entre ellas?
Por defecto, cuando un contenedor se elimina, sus datos también se pierden (_capa de contenedor_). Existen dos maneras para evitarlo[^6]:
- **Volumes**: monta el directorio en una ruta interna de Docker (administrados directamente por la API de Docker). Los volúmenes son compatibles con contenedores de diferentes SO. Como ventaja se tiene que compartir volúmenes entre varios contenedores es más seguro.  
- **Bind mounts**: permite montar un archivo o directorio desde el _host_ en el contenedor el que es accesible por un path absoluto pero dependen del SO y el filesystem del host. Es beneficioso cuando se usa al momento de desarrollar ya que se puede montar un bind mount al código fuente por lo tanto el contenedor ve los cambios que se realizan en el código de forma inmediata[^7]. 

| Característica | **Volúmenes**                           | **Bind Mounts**                         |
| -------------- | --------------------------------------- | --------------------------------------- |
| Gestión        | Docker los gestiona automáticamente     | El usuario gestiona la ruta manualmente |
| Portabilidad   | Más portables (independientes del host) | Menos portables (dependen del sistema)  |
| Seguridad      | Más seguros (aislados del host)         | Menos seguros (acceso directo al host)  |
| Rendimiento    | Mejor rendimiento en Docker Desktop     | Puede ser más lento o variable          |
| Uso típico     | Producción, bases de datos              | Desarrollo, acceso directo a archivos   |

## Taller
**El siguiente taller le guiará paso a paso para la construcción de una imagen Docker utilizando dos mecanismos distintos para los cuales deberá investigar y documentar qué comandos y argumentos utiliza para cada caso.**

1. Instale Docker CE (Community Edition) en su sistema operativo. Ayuda: seguir las instrucciones de la página de Docker. La instalación más simple para distribuciones de GNU/Linux basadas en Debian es usando los repositorios.

> Usaremos el SO otorgada en la cátedra de redes del año pasado equisde.

2. Usando las herramientas (comandos) provistas por Docker realice las siguientes tareas:
- Obtener una imagen de la última versión de Ubuntu disponible. ¿Cuál es el tamaño en disco de la imagen obtenida? ¿Ya puede ser considerada un contenedor? ¿Qué significa lo siguiente: _Using default tag: latest_?

Mediante el comando traemos una imagen desde DockerHub. Al no especificar la versión nos trae la última versión estable [^8].  
```bash
docker pull ubuntu
```
Aplicación: 
```bash
redes@debian:~$ docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
0622fac788ed: Pull complete 
Digest: sha256:6015f66923d7afbc53558d7ccffd325d43b4e249f41a6e93eef074c9505d2233
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```
Con el comando `docker images` podemos observarel listado de imagenes que tenemos con el detalle
```bash
redes@debian:~$ docker images
REPOSITORY                                  TAG       IMAGE ID       CREATED        SIZE
ubuntu                                      latest    a0e45e2ce6e6   2 weeks ago    78.1MB
hello-world                                 latest    74cc54e27dc4   3 months ago   10.1kB

```
No se puede considerar un contenedor porque para eso es necesario levantar la imagen con `docker run`

- De la imagen obtenida en el punto anterior iniciar un contenedor que
simplemente ejecute el comando `ls -l`.
Levantamos el contenedor y abrimos la shell. 
```bash
# docker run -it ubuntu bash
redes@debian:~$ docker run -it ubuntu bash
root@2e570bd38b92:/# ls -l
total 48
lrwxrwxrwx   1 root root    7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 22  2024 boot
drwxr-xr-x   5 root root  360 May 17 02:26 dev
drwxr-xr-x   1 root root 4096 May 17 02:26 etc
drwxr-xr-x   3 root root 4096 Apr 15 14:11 home

```

- ¿Qué sucede si ejecuta el comando `docker [container] run ubuntu /bin/bash`?
Si la imagen no existe en el sistema local lo descarga desde DockerHub y crea un nuevo contenedor a partir la imagen. El comando ejecuta `/bin/bash` dentro del contenedor.  

> aksjdnaks yo ejecuté con la shell interactiva de una por la documentación pero en realidad lo que se busca es ver que se ejecuta la shell pero al faltar el flag `-it` no se pueda interactuar con él, entonces el contenedor se detiene inmediatamente luego de iniciarse. 

- ¿Cuál es el PID del proceso bash en el contenedor? ¿Y fuera de éste?
Obtenemos el PID con `echo $$`. Dentro del contenedor nos imprime `1`.  
Fuera del contenedor aplicando: `docker inspect --format '{{.State.Pid}}' 92ff8c98a711 <CONTAINER_ID>` nos sale `3250`. 
Son diferentes porque el contenedor tiene su propio espacio de procesos aislado (namespace)

- Ejecutar el comando `lsns`. ¿Qué puede decir de los namespace?
```bash
root@92ff8c98a711:/# lsns
        NS TYPE   NPROCS PID USER COMMAND
4026531834 time        2   1 root bash
4026531837 user        2   1 root bash
4026532167 mnt         2   1 root bash
4026532168 uts         2   1 root bash
4026532169 ipc         2   1 root bash
4026532170 pid         2   1 root bash
4026532171 cgroup      2   1 root bash
4026532173 net         2   1 root bash
```
Todos los namespaces tienen el mismo PID que es el del bash del contenedor. 
Si ejecuto `lsns` desde el lado del host me salen diferentes id de namespaces lo que me confirma que el contenedor está correctamente aislado. 

- Dentro del contenedor cree un archivo con nombre sistemas-operativos en el directorio raíz del filesystem y luego salga del contenedor (finalice la sesión de Bash utilizando las teclas Ctrl + D o el comando `exit`).
- Corrobore si el archivo creado existe en el directorio raíz del sistema operativo anfitrión (host). ¿Existe? ¿Por qué?
No existe porque los contenedores tienen un sistema de archivos aislados del host, desaparece cuando el contenedor se elimina.

- Vuelva a iniciar el contenedor anterior utilizando el mismo comando (con una terminal interactiva). ¿Existe el archivo creado en el contenedor? ¿Por qué?
Por los mismos motivos que se explicaron antes y al inicio la capa de escritura se genera con cada contenedor y es efímero, por lo tanto no persiste los datos a menos que usemos volumes o bind mounts. 
```bash
root@92ff8c98a711:/# touch sistemas-operativos.txt
root@92ff8c98a711:/# ls
bin   dev  home  lib64  mnt  proc  run   sistemas-operativos.txt  sys  usr
boot  etc  lib   media  opt  root  sbin  srv                      tmp  var
root@92ff8c98a711:/# exit
exit
redes@debian:~$ docker run -it ubuntu bash
root@ca3073f34b64:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

/ Obtenga el identificador del contenedor (container_id) donde se creó el archivo y utilícelo para iniciar con el comando `docker start -ia container_id` el contenedor en el cual se creó el archivo.
i. ¿Cómo obtuvo el container_id para para este comando?
Mediante `docker ps -a` que lista contenedores que existen aunque estén detenidos.
ii. Chequee nuevamente si el archivo creado anteriormente existe. ¿Cuáles el resultado en este caso? ¿Puede encontrar el archivo creado?

```bash
redes@debian:~$ docker start -ia 92ff8c98a711
ls
bin   dev  home  lib64  mnt  proc  run   sistemas-operativos.txt  sys  usr
boot  etc  lib   media  opt  root  sbin  srv                      tmp  var
```
El archivo creado existe, Docker guarda el sistema de archivos del contenedor incluso si está detenido a menos que se lo elimine explícitamente con `docker rm <container_id>`

- ¿Cuántos contenedores están actualmente en ejecución? ¿En qué estado se encuentra cada uno de los que se han ejecutado hasta el momento?
Con `docker ps` se muestran los contenedores en ejecución. 
- Elimine todos los contenedores creados hasta el momento. Indique el o los comandos utilizados.
Con `docker rm $(docker ps -aq)` se eliminan todos los contenedores pero como tengo virtualizado hay contenedores que mantienen el sistema así que manualmente con el ID de los contenedores lo eliminaremos con el mismo comando (`docker rm <container_id>`). 


[^1]: https://www.redhat.com/es/topics/containers/what-is-docker
[^2]: https://www.ibm.com/mx-es/think/topics/docker
[^3]: https://www.ibm.com/think/topics/containers?_gl=1*h54ii8*_ga*MTM5NzIzOTE0My4xNzEyMDg5OTgy*_ga_FYECCCS21D*MTc0NTk0OTAzOC4yMDkuMC4xNzQ1OTQ5MDM4LjAuMC4w
[^4]: https://www.ibm.com/mx-es/think/topics/docker#:~:text=comandos%20de%20Docker.-,Im%C3%A1genes%20de%20Docker,-Las%20im%C3%A1genes%20de
[^5]: https://interlir.com/es/2024/11/11/gestion-de-direcciones-ip-en-docker-y-kubernetes/
[^6]: https://medium.com/@williehung/persisting-data-using-docker-volume-and-bind-mount-52a8cb42f4f0
[^7]: https://medium.com/@williehung/persisting-data-using-docker-volume-and-bind-mount-52a8cb42f4f0
[^8]: https://docs.docker.com/reference/cli/docker/image/pull/