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


Una vez instalado comprobamos que el servicio esta activo. No hay que olvidar que el servicio nginx hace uso  del puerto 80, asi que 
primero tendremos que desactivar cualquier otro servicio que pueda estar haciendo uso de dicho puerto.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_2.png)


La configuración de nginx para que funcione como un balanceador de carga entre los dos servidores web deberemos editar el archivo **/etc/nginx/conf.d/default.conf** con la siguiente configuración.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_3.png)


El archivo de configuración se divide en:

  - **upstream**: Aqui definiremos un nombre para el grupo de servidores que vamos a balancear. Por cada servidor añadiremos la siguiente sintaxis es **server [ip_maquina objetivo]**.

  - **server**: En esta sección nos encargaremos de configurar propiamente el funcionamiento del balanceador, añadiendo el upstream que debe usar, el protocolo o como debe editar la cabecera Connection. También se define la ruta hacia los log de nginx.

Ahora comprobamos el funcionamiento de nginx usando el comando curl lanzando una petición a la ip **192.168.56.4**


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_6.png)


Vemos que algo no funciona, ya que nos muestra el index.html que habíamos definido para el balanceador de carga. Esto puede ocurrir si en el archivo de configuración **/etc/nginx/nginx.conf** esta descomentada la linea **include /etc/nginx/sites-enabled/**. Comentamos la linea y reiniciamos el servicio nginx.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_5.png)


Vamos a intentar de nuevo conseguir una petición de los servidores finales pasando a través del balanceador.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_7.png)


Vemos que si que nos esta balanceando correctamente entre un servidor y otro. Por defecto, el algoritmo de balanceo usado es Round-Robin. Si quisieramos usar por ejemplo un algoritmo basado en ponderación, tendríamos que añadir al archivo de configuración **default.conf** lo siguiente.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_9.png)


Aqui estamos definiendo sobre la ip **192.168.56.2** el doble de "peso" que sobre la ip **192.168.56.3**. Así, la primera máquina recibira de media el doble de peticiones que la segunda máquina.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_10.png)

## Instalación de HAProxy

Haproxy será el segundo balanceador que instalaremos. Para ello haremos uso del comando **apt-get** de nuevo.


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_11.png)


Al igual que con nginx tendremos que editar el archivo de configuración **/etc/haproxy/haproxy.cfg**. Añadiremos la siguiente configuración:


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_13.png)


En el archivo configuración reservamos el puerto 80 para esperar las conexiones entrantes y definimos el nombre del grupo de servidores que vamos a usar. En este caso servers define el grupo de servidores, en el cual estan los dos servidores apache que tenemos sirviendo paginas web. Por cada uno añadimos la ip y el puerto. 

Reiniciamos el servicio HAProxy y comprobamos que balancee correctamente las peticiones.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_15.png)

## Comparación: Nginx vs HAProxy

En esta sección vamos a realizar una comparativa de tiempos entre los dos balanceadores de carga frente a **20000** peticiones en grupos de **500** de forma concurrente. Para ello haremos uso de la herramienta **Apache Benchmark**. Primero sobrecargaremos el balanceador Nginx. A continuación vemos el rendimiento del servidor con los parametros de carga mencionados antes:


![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_16.png)

Ahora paramos el servicio **nginx** y arrancamos el servicio **haproxy**. Volvemos a lanzar el comando **ab**.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica3/img/swap3_17.png)

Viendo el analisis de apache benchmark en cada caso podemos ver:

  - Número de peticiones por segundo: en el caso de **nginx**, el número de peticiones por segundo que ha servido es de 
**1390.97** frentre a las **2576.25** peticiones de HAProxy. Este último ha conseguido servir casi el doble de peticiones por segundo.
  - Tiempo medio de respuesta por petición: para **nginx**, la media ha sido de **359.462 ms** y **0.719 ms** para peticiones dentro del mismo grupo concurrente. En cambio para **haproxy** la media ha sido de **194.081 ms** y **0.388 ms** dentro del grupo concurrente. De nuevo podemos observar que **haproxy** supera en tiempo de respuesta velocidad a nginx.
  
