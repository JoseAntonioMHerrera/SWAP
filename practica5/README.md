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


## Jerarquía Maestro-Esclavo

El primer paso es habilitar el módulo de apache de ssl, usando la orden **a2enmod**.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_1.png)

Este módulo permitirá a apache usar las conexiones encriptadas con el protocolo ssl. Tras activar el módulo, debemos apagar y volver a encender el servicio de apache. Para la creación del certificado autofirmado vamos a usar la herramienta **open-ssl**.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_2.png)

La sintaxis del comando es la siguiente:

  - **req -x509**: petición de creación de un certificado (Certificate Signing Request). La opción -x509 representa el estandar para infraestructuras de clave pública, esto es certíficados con clavés públicas.
  
  - **-nodes**: si se especifica y una clave privada fuera a ser creada, esta no se encriptaría.
  
  - **-days n**: si se usa, establece el número de días que el certificado será valido como tal.
  
  - **-newkey rsa:2048**: crea un certificado y una nueva clave privada. El segundo parametro establece el tipo de clave y el número de bits de la clave.
  
  - **-keyout [path]**: ruta donde se guardará la nueva clave privada recién creada.
  
  - **-out [path]**: ruta donde se guardará el nuevo certificado recién creado.
  
Cuando ejecutamos el comando anteriormente explicado, se nos pedirá que rellenemos los datos del certificado. Estos son tales como país y provincia, institución a la que representa el certificado, etc... En nuestro caso hemos rellenado todo con swap, salvo el correo electrónico que será el mío propio.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_3.png)

Tanto la clave privada como el certificado deberán ser el mismo tanto en los servidores web como en el balanceador. Se recomienda el uso de alguna herramienta como **rsync** para copiar estos dos archivos en el resto de máquinas. Una vez hecho esto, y suponiendo que hayamos habilitado también el módulo ssl también en la segunda máquina servidora, vamos a proceder a editar el archivo */etc/apache2/sites-avaible/default-ssl.conf*. Aquí añadiremos la ruta a nuestro certificado autofirmado.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_4.png)

Un último paso será activar el archivo *default-ssl.conf* usando la herramienta **a2ensite**. Esta orden requiere permisos de usuario.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_5.png)

Ahora al realizar una petición a la ip **192.168.56.2** usando el protocolo https nos aparecerá un aviso al intentar acceder a un sitio con un certificado autofirmado.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_6.png)

Este mismo proceso lo aplicaremos en el segundo servidor apache de la máquina 2. Por otra parte, en el servidor balanceador, vamos a realizar la configuración de nginx. Para ello, en el archivo */etc/nginx/conf.d/default.conf*. El archivo debe quedar de la siguiente manera:


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_9.png)


De nuevo, accediendo usando el protocolo seguro a la ip **192.168.56.4** (la ip del balanceador) nos saldrá la misma advertencia que en el paso anterior.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_10.png)

##Protegiendo el servidor: firewall

En esta sección vamos a mejorar la seguridad de uno de los servidores finales aplicando reglas firewall para permitir solo la entrada y salida por los puertos 22, 80 y 443. Además, permitiremos todas las conexiones desde localhost. Como paso previo, primero borraremos todas las reglas del firewall para que solo esten activas las hemos mencionado anteriormente. El comando a usar es **iptables**.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_17.png)

Para hacer estas reglas persistentes vamos a crear un servicio que ejecute el script de la imagen anterior al arrancar el servidor. Para ello nos haremos uso de **systemd**. Primero crearemos el servicio que se encargará de ejecutar el script:

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_21.png)

Si el script (que aqui ubicamos en la carpeta */sbin/*) no tuviera permisos para su ejecución habría que proporcionarselos. Una vez creado el servicio, lo activamos con **systemctl** y hacemos que se ejecute al arrancar el sistema con la opción **enable**. 

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica4/img/SWAP4_18.png)
