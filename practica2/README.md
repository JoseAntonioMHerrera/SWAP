# Práctica 2 - SWAP
## Introducción

En esta práctica veremos como clonar archivos de una máquina a otra, con el objetivo de mantener una duplicación de datos que se mantenga 
actualizada a lo largo del tiempo de forma automática (esto es, sin acción directa del administrador). Primero Para poder realizar esta tarea con
éxito tendremos que configurar el ssh para que pueda conectarse de una máquina a otra sin que nos solicite la contraseña.

## Copia de archivos manual

En esta sección vamos a realizar un copiado manual del directorio */var/www/html*. Probaremos dos opciones distintas: en la primera
comprimiremos la carpeta haciendo uso del comando **tar** y la enviaremos mediante ssh a la máquina objetivo. La segunda opción ahce uso comando es **scp**, el cual es una versión del comando
cp que nos permite copiar archivos de forma segura remotamente. A continuación se muestra el resultado de estos dos comandos.

Ejecutamos el comando **tar** en la máquina 1 (**192.168.56.2**) y mediante un pipeline se lo pasamos al comando ssh que establece la conexión con la máquina 2 (**192.168.56.3**)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_1.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica2/img/pract_2_swap_2.png)

