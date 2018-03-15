# Práctica 1
##### Alejandro Manuel Hernández Recio y Francisco Miguel Toledo Aguilera

## Escenario
Hemos utilizado VirtualBox en host Ubuntu 16.04, con dos máquinas virtuales Ubuntu Server 16.04 conectadas ambas a un adaptador *host only*.

Las IP son las siguientes:
 - Máquina 1 (P1M1): 192.168.56.101
 - Máquina 2 (P1M2): 192.168.56.102
![Configuración de red de las máquinas](https://i.imgur.com/1hrKknz.png)

## Desarrollo
Lo primero que hicimos fue activar la cuenta de superusuario:
```sh
$ sudo
# passwd
```
Tras esto, instalamos los paquetes para montar el servidor LAMP:
```sh
sudo apt-get install apache2 mysql-server mysql-client
```
Y comprobamos que Apache está ejecutándose:
```sh
ps aux | grep apache
```
![](https://imgur.com/udts8x4.jpg)

En este caso, nosotros ya teníamos cURL instalado, así que pasamos directamente a meter en el servidor web las páginas de prueba para poder acceder. Hay que ubicarlas en ``/var/www/html/``.

### Objetivos de la práctica

**1. Acceder por ssh de una máquina a otra**

Conexión de P1M1 a P1M2:

![](https://imgur.com/yLIBVxV.jpg)

Conexión de P1M2 a P1M1:

![](https://imgur.com/BbdEVU5.jpg)

**2. Acceder mediante la herramienta curl desde una máquina a la otra**

![](https://imgur.com/VM0WmDv.jpg)
