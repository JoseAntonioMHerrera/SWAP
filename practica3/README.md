# Práctica 3 - SWAP
## Introducción

En esta práctica vamos a configurar dos balanceadores de carga, nginx y haproxy, que repartan la carga entre dos servidores web apache que hemos
configurado en anteriores practicas. Para ello Procederemos a su descarga y configuración. También realizaremos una comparativa entre
estos dos balanceadores cuando son sometidos a una carga alta de peticiones, usando para ello la herramienta ab que simulará una cantidad
lo suficientemente importante de peticiones sobre los dos balanceadores de carga.

## Instalación de Nginx

En esta sección vamos a realizar la instalación, configuración y prueba del balanceador de carga Nginx. Para la instalación recurriremos
al comando apt-get. La nueva máquina tendra asignada la ip **192.168.56.4** siendo la de los servidores web **192.168.56.2** y
**192.168.56.3**.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_1.png)


Una vez instalado comprobamos que el servicio esta activo. No hay que olvidar que el servicio nginx hace uso  del puerto 80, asi que 
primero tendremos que desactivar cualquier otro servicio que pueda estar haciendo uso de dicho puerto.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_2.png)


La configuración de nginx para que funcione como un balanceador de carga entre los dos servidores web deberemos editar el archivo **/etc/nginx/conf.d/default.conf**
con la siguiente configuración.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_3.png)


El archivo de configuración se divide en:
  - **upstream**: Aqui definiremos un nombre para el grupo de servidores que vamos a balancear. La sintaxis es **server [ip_maquina objetivo]**

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


## Automatizando las tareas: crontab

Como paso final para poder automatizar el clonado de directorios entre máquinas, vamos a añadir una nueva tarea que se ejecute al inicio de cada hora para que mantenga los directorios actualizados. Para ello vamos a editar el archivo */etc/crontab* en la máquina 2 añadiendo la última linea de la siguiente imagen.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_14.png)

Los primeros cinco parametros de la linea reflejan los minutos, horas, día del mes, mes y día de la semana. los cuales marcarán la rutina de activación de la tarea. En nuestro caso ponemos un 0 en los minutos, y en el resto añadimos un * para indicar todos los posibles valores. La orden es la misma que hemos visto en secciones anteriores.
