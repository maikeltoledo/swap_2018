# Práctica 3
##### Alejandro Manuel Hernández Recio y Francisco Miguel Toledo Aguilera

## Escenario
Hemos utilizado VirtualBox en host Windows 10, con tres máquinas virtuales Ubuntu Server 16.04 y un cliente Windows 7, todos conectados a un adaptador *host only*.

La topología sería la siguiente:

![](https://i.imgur.com/vbTaXlg.png)

## Desarrollo 
### Instalar nginx en Ubuntu Server
Pasamos a configurar Balanceador:

``sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove``

``sudo apt-get install nginx ``

``sudo systemctl start nginx``

Podemos comprobar que se ha instalado mostrando la versión:

![](https://i.imgur.com/JEGX3sy.png)

###  Balanceo de carga usando nginx 

Pasamos a su configuración modificando el fichero ``/etc/nginx/conf.d/default.conf``

![](https://i.imgur.com/6rFJVB0.png)

Guardamos los cambios y procedemos a modificar el fichero de configuración nginx.conf. Comentamos la línea marcada en amarillo para que solamente utilice nuestro default.conf (el fichero anterior):

![](https://i.imgur.com/GYdWB27.png)

Reiniciamos el servicio:

``root@balanceador:~# service nginx restart;``

Ahora probamos en el cliente, accediendo a la IP del balanceador:

![](https://i.imgur.com/kMTEHt7.png)


###  Balanceo de carga con haproxy 

En este paso debemos de detener el servicio para probar otro balanceador de carga y no inferir en el mismo puerto:

``root@balanceador:~# service nginx stop;``

Instalamos haproxy: 

``root@balanceador:~# apt-get install haproxy``

y procedemos a su configuración:


``root@balanceador:~# nano /etc/haproxy/haproxy.cfg``

![](https://i.imgur.com/Smbr2TC.png)

Lanzamos el servicio haproxy:

``root@balanceador:~# /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg``

Y volvemos a hacer la prueba con nuestro cliente:


![](https://i.imgur.com/dj0oXnp.png)