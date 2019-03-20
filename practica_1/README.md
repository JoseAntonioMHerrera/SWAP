# Práctica 1 - SWAP_2019
## Introducción

En esta práctica se nos propone como objetivo que pongamos en marcha una serie de herramientas con las que trabajaremos en las siguientes prácticas. La elección de las herramientas a utilizar son las siguientes:
	
- Ubuntu server 16.04
- Servidor LAMP (Linuz, Apache, MySQL y Php)
- Servidor y cliente ssh

A continuación se mostrará el proceso de instalación y el correcto funcionamiento de dichas herramientas.

## Instalación y configuración de Ubuntu Server 16.04

La instalación de la máquina virtual Ubuntu 16.04 se ha realizado siguiendo los pasos por defecto de la instalación. Para la segunda máquina he optado por clonar la máquina instalada. Con el objetivo de crear una red local donde pueda trabajar con las dos máquinas y el host he creado con ayuda de la interfaz de virtual box con ip **192.168.56.0**. Se han editado el archivo */etc/network/interfaces* de ambas máquinas para asignarles una ip estática dentro de la red (**192.168.56.2** y **192.168.56.3** respectivamente).

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_5.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_4.png)

Comprobamos que tenemos conectividad entre las dos máquinas haciendo uso del comando ping.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_6.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_7.png)

## Instalación y configuración de Apache2

Para la instalación de Apache (junto a MySQL y Php) hemos seleccionado la opción correspondiente dentro del proceso de instalación de Ubuntu.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_1.png)

Para comprobar que el servicio funciona correctamente vamos a realizar una petición desde una máquina al servidor web de la otra y viceversa usando para ello el comando Curl para comprobar que tenemos un correcto acceso entre las máquinas.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_8.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_9.png)


## Instalación y configuración de Ssh

Para la instalación de openssh-server hemos seleccionado junto a la instalación del servidor LAMP la opción de instalar el servidor OpenSSH, el cual nos instalá el servidor necesario para poder conectarnos a la máquina usando ssh. Comprobamos que podamos acceder desde las dos máquinas mediante ssh.

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_10.png)

![image](https://github.com/JoseAntonioMHerrera/SWAP_2019/blob/master/practica_1/img/pract_1_swap_11.png)
