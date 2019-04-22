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
  
