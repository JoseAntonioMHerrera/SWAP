# Práctica 5 - SWAP
## Introducción

En esta práctica vamos a ver varias opciones para el mantenimiento de una base de datos replicada en un segundo servidor. Los primeros acercamientos consistirán en un replicado manual, extrayendo la base de datos elegida, copiandola en la máquina destino y reconstruyendola allí. Después refinaremos este proceso automatizándolo con una jerarquía maestro-esclavo. 

## Mysql: creación de tablas e introducción de datos

Primeramente vamos a crear una tabla y a introducir una fila para poder tener algo que replicar en la máquina destino.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_1.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_2.png)

En la siguiente sección vamos a proceder a volcar la base de datos a la que hemos llamado contactos.

## Mysqldump: volcado de la base de datos

El comando que usaremos para volcar la base de datos será **mysqldump**. Hay que tener en cuenta que durante el proceso de replicación tendríamos que asegurarnos de que durante ese tiempo no se ha insertado ni eliminado nada. Sería una buena idea bloquear las tablas primero.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP_5_3.png)

Ahora ejecutamos el comando **mysqldump** sobre la base de datos *contactos* y guardamos el fichero resultante en la carpeta */tmp*

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_4.png)

Una vez que hemos volcado la base de datos podemos proceder a desbloquear las tablas.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/swap5_5.png)

Para pasar el archivo de la máquina 1 a la máquina 2 nos iremos a esta última y ejecutaremos el comando **scp**. Nos permite realizar una copia de archivos entre distintos hosts a traves de ssh.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP_5_6.png)

Una vez que tenemos el archivo en la máquina destino tan solo tendríamos que hacer el paso contrario al que hemos hecho para hacer el dump de la base de datos. Como paso previo debemos de crear la base de datos ya que el archivo *.sql* que tenemos tiene las sentencias sql necesarias para reconstruir la base de datos, pero no para crearla.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_7.png)


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_8.png)

## Jerarquía Maestro-Esclavo

La jerarquía Maestro-Esclavo establece una máquina como maestro y otra o varias otras como esclavo. Así, los cambios hechos en el maestro se verá reflejado en los esclavos. Esta configuración tiene el beneficio de la automatización del proceso de replicar una base de datos en el servidor de backup. Para usar esta configuración primero vamos a editar el archivo */etc/mysql/mysql.conf.d/mysqld.cnf* y editaremos las siguientes lineas.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_9.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_10.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_11.png)

Hemos descomentado (o añadido en su defecto) la linea que nos permite recibir conexiones desde localhost, la ruta hacia los archivos donde se almacenarán los logs y la id del servidor. Este último campo variará a la hora de configurar la máquina esclavo, dandole un id distinto. En nuestro caso le daremos el id 2.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_14.png)

Guardamos los ficheros y reiniciamos mysql en ambas máquinas.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_13.png)

Ahora crearemos en la máquina maestro el usuario que la máquina esclavo usará para conectarse a la base de datos y así replicar el contenido elegido.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_15.png)

Ahora mostramos información de los archivos de log binarios de la máquina maestro que mas adelante usaremos para realizar la configuración en mysql de la parte del esclavo.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_16.png)

En el esclavo, vamos a cambiar el maestro por la ip de la máquina 1, proporcionandole también el usuario que acabamos de crear así como información sobre los logs de la máquina maestro. Después lanzamos el esclavo.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_17.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica5/img/SWAP5_18.png)
