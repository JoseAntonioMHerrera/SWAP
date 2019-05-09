# Práctica 4 - SWAP
## Introducción

En esta práctica vamos a configurar un certificado autofirmado para usarlo en los dos servidores web finales y en el balanceador, con el fin de que los usuarios puedan conectarse mediante https. Por otra parte, en uno de los servidores web finales vamos a crear un servicio que se ejecute en el arranque del servidor y que configure unas reglas de firewall que solo permitan el acceso mediante ssh y acepte peticiones en los puertos 80 y 443 (tanto peticiones entrantes como salientes) y bloquee el resto de tráfico.

## Creación de un certificado autofirmado

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
