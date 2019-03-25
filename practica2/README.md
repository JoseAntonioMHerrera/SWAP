# Práctica 2 - SWAP
## Introducción

En esta práctica veremos como clonar archivos de una máquina a otra, con el objetivo de mantener una duplicación de datos que se mantenga 
actualizada a lo largo del tiempo de forma automática (esto es, sin acción directa del administrador). Para poder realizar esta tarea con
éxito tendremos que configurar el ssh para que pueda conectarse de una máquina a otra sin que nos solicite la contraseña.

## Copia de archivos manual

En esta sección vamos a realizar un copiado manual del directorio */var/www/html*. Probaremos dos opciones distintas: en la primera
comprimiremos la carpeta haciendo uso del comando **tar** y la enviaremos mediante ssh a la máquina objetivo. La segunda opción ahce uso comando es **scp**, el cual es una versión del comando
cp que nos permite copiar archivos de forma segura remotamente. A continuación se muestra el resultado de estos dos comandos.

Ejecutamos el comando **tar** en la máquina 1 (**192.168.56.2**) y mediante un pipeline se lo pasamos al comando ssh que establece la conexión con la máquina 2 (**192.168.56.3**)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_1.png)

Aqui vemos como en la carpeta *home/* del usuario de la segunda máquina se ha copiado el archivo tar.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_2.png)


## Clonado de archivos con rsync

En este apartado vamos a realizar un clonado del directorio *www/* de la máquina 1 desde la máquina 2 para que su contenido sea exactamente el mismo. En el caso de que no lo tuvieramos instalado, lo instalamos usando el comando **sudo apt-get install rsync**. Otra cosa que deberíamos realizar para el correcto funcionamiento del ejercicio, si no estamos usando el usuario root es establecer los permisos apropiados para poder manejar el borrado y la edición.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_4.png)

El comando que vamos a usar se llama **rsync** y consta de los siguientes parametros:

  - **-avz**: Respectivamente indican que se trata de archivos, verbose y que se comprima para la transferencia a la máquina objetivo.
  - **-e**: permite seleccionar la shell remota que queremos usar con el comando rsync. En nuestro caso será ssh.
  - **[ip_objetivo]@[directorio_objetivo] [directorio]**: indicamos la ip de la máquina objetivo y seguido de @ el directorio objetivo a clonar. Por último, el directorio donde queremos clonarlo.
  
  A continuación se muestran imagenes mostrando el resultado del comando rsync.
  
![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_5.png)

En el siguiente caso hemos hecho uso de la opción **--delete** la cual nos permite además de mantener los archivos actualizados mantener también la estructura del directorio, eliminando los ficheros o directorios que se hayan eliminado en la máquina objetivo. Otra opción que se ha incluido es la de **--exclude** pudiendo así seleccionar una serie de directorios/ficheros con la opción **--exclude=/[directorio]** los cuales no se clonarán.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_8.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_9.png)


## Indetificación ssh mediante claves pública/privada

Vamos a configurar la identificación en ssh mediante claves pública y privada desde la máquina 2 a la máquina 1 para no tener que introducir la contraseña cada vez que queramos acceder mediante ssh. Para ello vamos a usar los siguientes comandos:

  - **ssh-keygen**: usaremos este comando para generar las claves pública y privada. Las opciones **-b** y **-t**  nos permitirán, respectivamente, seleccionar el número de bits de la clave a crear y el típo de key a crear.
  - **ssh-copy-id**: este comando nos permite copiar nuestra clave pública en el servidor destino. Esta clave se copiará al archivo *~/.ssh/authorized_keys*. Si no existiera, se crearía en el momento de la copia.
  
  A continuación se muestra el creado de claves y la copía de la clave pública.
  
![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_10.png)
  
![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_11.png)

En la carpeta *~/.ssh/* se ha creado el archivo *authorized_keys*

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_12.png)

Ahora, podemos acceder mediante ssh sin tener que introducir la contraseña.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_13.png)


## Automatizando las tareas: Crontab
