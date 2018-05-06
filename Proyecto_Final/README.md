![Imgur](https://i.imgur.com/9Hh6pP2.png)
# Autores
##### Francisco Miguel Toledo Aguilera y Alejandro Manuel Hérnandez Recio

# Introducción

Raspberry Pi es un pequeño ordenador con prestaciones decentes y con un sinfín de usos. Dado su bajo precio, se está implementando en escuelas y pequeñas pymes. Nosotros vamos a realizar una granja web utilizando estos equipos. Tendremos configurados dos servidores web replicados con su correspondiente balanceador, un servidor de base de datos, y un servidor de ficheros ownCloud.

## Hardware empleado

 * Cinco Raspberry Pi
	 * Dos RPi 1 Model B.
	 * Dos RPi 2 Model B.
	 * Una RPi 3 Model B.
* Un switch
 * Router fibra

## Topología
![Imgur](https://i.imgur.com/pGehuR1.jpg)
![Imgur](https://i.imgur.com/jnfl92y.jpg)

##### Nombres de equipos, direcciones IP y SO utilizado

|Hostname | IP | SO | Modelo |  
| -- | -- | -- | -- |
| balanceador | 192.168.1.10 | DietPi | RPi 1
| web1 | 192.168.1.11 | Raspbian 9 | RPi 2
| web2 | 192.168.1.12 | DietPi | RPi 1
| nube | 192.168.1.13 | Raspbian 9 | RPi 3
| bdserver | 192.168.1.14 | Raspbian 9 | RPi 2


# Apertura de puertos y configuración de No-ip
Para poder sacar nuestro servicio a internet, es necesario o bien tener contratado un VPS (Virtual Private Server), o bien una IP fija y configurar la NAT. El servicio de VPS es incompatible con nuestro objetivo de  montar una granja web con equipo real, y la IP fija tiene un coste añadido de 15€ + IVA. La solución es entonces utilizar **DDNS** (Dynamic DNS). Hay proveedores gratuitos, como ddns.net, que es el que hemos utilizado nosotros. El dominio es toledoaguilera.ddns.net.

Para que el DDNS funcione, hay que tener una manera de *decirle* a ddns.net que nos han cambiado la IP. Aquí entra en juego el cliente no-ip, que vamos a proceder a configurar en el siguiente apartado. 

## Cliente No-ip
El cliente de No-ip, se puede configurar de varias formas, una de ellas es instalando un proceso en cualquiera de los servidores para que actualice cada X tiempo la información sobre nuestra IP pública. En nuestro caso, hemos decidido utilizar el apartado de configuracíon de No-ip en el router, donde hay que definir el dominio e introducir nuestro usuario y contraseña de No-ip:
![Imgur](https://i.imgur.com/PiYf24F.jpg)

## Apertura de puertos (NAT)
Como bien ya sabemos, la NAT asocia un puerto externo a una IP  interna y un puerto interno. Esto hace posible que bajo una única IP pública, se pueda acceder a diferentes equipos que están ejecutando diferentes aplicaciones, cada uno con una IP privada distinta.

Los puertos que vamos a abrir nosotros son:

 - SSH (22). Abierto para el balanceador. Desde esta máquina configuraremos los demás equipos de la granja desde el exterior.
 - HTTPS (443). Abierto para el balanceador. Sólo vamos a permitir tráfico seguro.
 - ownCloud (9090). Abierto para nube. Desde este puerto se accede a la interfaz web de ownCloud.

![Imgur](https://i.imgur.com/p78RnGU.jpg)

# Servicio web
En esta sección vamos a describir cómo hemos montado e instalado el servicio web de nuestra granja. Hablaremos de la configuración HTTPS con **certificado válido** y el balanceador. 
## Instalación y configuración de Apache
### Apache y PHP 7
Para instalación de apache en los dos servidores, ejecutamos el siguiente comando en ambos:
```bash
sudo apt-get -y install apache2
```
También instalamos PHP:
```bash
sudo apt-get -y install php7.0 libapache2-mod-php7.0
```
Por último reiniciamos apache:
```bash
service apache2 restart
```
### Rsync
Ahora vamos a instalar una herramienta para la replica de información entre los dos servidores, para su instalación ejecutamos lo siguiente:
```bash
sudo apt-get install rsync
```
Cambiamos los permisos de /var/www/:
```bash
sudo chown user:user -R /var/www
```
Ahora nos toca configurar el acceso sin contraseña para ssh para posteriormente, configurar en el fichero contrab la sincronización de archivos.
En nuestra máquina secundaria (Apache 192.168.1.12) geeramos la clave:
```bash
ssh-keygen -b 4096 -t rsa
```
ahora copiamos la clave pública a la maquina principal (Apache 192.168.1.11), añadiéndola al fichero `~/.ssh/authorized_keys` que deberá tener los permisos 600:
```bash
chmod 600 ~/.ssh/authorized_keys
```
hacemos la copia de la clave:
```bash
ssh-copy-id 192.168.1.11
```
en este punto ya nos podemos conectar al equipo sin contraseña.
### Crontab
Para programar la tarea de copia en segundo plano desde la máquina principal a la secundaria, añadimos la siguiente linea al fichero de configuración de crontab:
`00 *    * * *   root    rsync -avz -e ssh root@192.168.1.11:/var/www/ /var/www/`

### Certificado SSL Let’s Encrypt
La instalación del certificado primero se hace sobre la máquina **web1** (192.168.1.11), instalamos git:
```bash
sudo apt-get -y install git
```
Nos vamos al siguiente directorio y clonamos el repositorio de letsencrypt:
```bash
cd /usr/local
```
```bash
sudo git clone https://github.com/letsencrypt/letsencrypt
```
#### Generar un certificado SSL para Apache
Dentro del directorio:
```bash
cd /usr/local/letsencrypt
```
Generamos el certificado con nuestro nombre de dominio con el siguiente comando:
```bash
./letsencrypt-auto --apache -d toledoaguilera.ddns.net
```
Y seguimos los pasos: 
- Acuerdo de licencia
- Introducimos un email válido
- Elegimos si los clientes pueden navegar usando HTTP y HTTPS o redirigir todas las solicitudes por HTTPS (En nuestro caso hemos seleccionado la redirección de todas las solicitudes por HTTPS).

Ya tenemos nuestro certificado instalado y se encuentra en:
```bash
/etc/letsencrypt/live/toledoaguilera.ddns.net/
```
#### Crear el sitio
Creamos nuestro sitio en `sites-available` donde le decimos donde se encuentran los certificados:
![Imgur](https://i.imgur.com/nOK65zk.jpg)
También es necesario cambiar el fichero de configuración de hosts del sistema, añadiendo a toledoaguilera.ddns.net:
![Imgur](https://i.imgur.com/TAanzI9.jpg)
Ahora habilitamos el ssl y nuestro sitio:
```bash
sudo a2enmod ssl
```
```bash
sudo a2ensite toledoaguilera.ddns.net
```
y reiniciamos el servicio:
```bash
service apache2 restart
```
#### Renovación automática del certificado
Los certificados son validos por 90 días, para renovarlos automáticamente, añadimos la siguiente línea a nuestro fichero crontab:
`0 1 1 */2 * cd /usr/local/letsencrypt && ./letsencrypt-auto certonly --apache --renew-by-default --apache -d toledoaguilera.ddns.net >> /var/log/toledoaguilera.ddns.net-renew.log 2>&1
`
Los detalles sobre la renovación del dominio, se encuentran en el archivo de configuración:
```bash
/etc/letsencrypt/renewal/toledoaguilera.ddns.net.conf
```
Ya tendríamos el certificado instalado en un servidor web.

### SSL en máquina secundaria
Como el dominio va a ser el mismo, basta con que copiemos los dos ficheros generados, que están en ``/etc/letsencrypt/live/toledoaguilera.ddns.net/`` de una máquina a la otra. Los pasos a configurar son los mismos que aparecen en [esta sección](#crear-el-sitio).

Ya tendríamos SSL en las dos máquinas.
## Instalación y Configuración de Nginx
Pasamos a configurar balanceador; ejecutamos los siguientes comandos:
```bash
sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove
```
```bash
sudo apt-get install nginx
```
```bash
sudo systemctl start nginx
```
### Balanceo de carga usando nginx
Modificamos el fichero de configuración `/etc/nginx/conf.d/default.conf`
![Imgur](https://i.imgur.com/KZKxrL1.jpg)

Guardamos los cambios y procedemos a modificar el fichero de configuración nginx.conf `/etc/nginx/nginx.conf` y comentamos la línea donde include sites-enabled puesto que solo vamos a utilizar nginx de balanceador:
![Imgur](https://i.imgur.com/AhaoFNS.jpg)
Una vez hecho estos cambios, reiniciamos el servicio:
```bash
service nginx restart;
```
En este punto ya tendríamos configurado nuestros dos servidores webs Apache junto con el balanceador de carga nginx.

# Servidor de bases de datos (MariaDB)
MariaDB viene como base de datos por defecto en Debian 9, así que para Raspbian pasa lo mismo.

Para configurarlo, lo único que necesitamos será crear las bases de datos para ownCloud y para Wordpress, el CMS que hemos usado para crear la página web principal. Además, para hacer las cosas bien, crearemos un usuario para el servicio de ownCloud y otro para el de Wordpress, cada uno restringido a su base de datos.

*Nota:* las contraseñas se hashean mediante la función PASSWORD(). Para obtener el hash, se puede hacer:
```sql 
				SELECT PASSWORD('tu_contraseña');
```
o bien usar esta web: https://www.browserling.com/tools/mariadb-password

**ownCloud**
```sql
mariadb> CREATE DATABASE owncloud;
mariadb> CREATE USER 'owncloud'@'%' IDENTIFIED BY '*HASH'; 
mariadb> GRANT ALL PRIVILEGES ON owncloud.* TO owncloud@'%' IDENTIFIED BY '*HASH';
mariadb> GRANT ALL PRIVILEGES ON owncloud.* TO owncloud@localhost IDENTIFIED BY '*HASH';
```

**WordPress**
```sql
MariaDB [(none)]> CREATE DATABASE wordpress;
MariaDB [(none)]> CREATE USER 'wordpress'@'%' IDENTIFIED BY '*HASH';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@'%' IDENTIFIED BY '*hash';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@localhost IDENTIFIED BY '*HASH';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Íbamos a configurar la RPi de nube como servidor de BD también y hacer replicación maestro-maestro, pero va bastante apurada de recursos corriendo solamente el ownCloud.

# ownCloud
ownCloud es un servidor que permite montar un sistema de almacenamiento de ficheros al estilo de Dropbox o Google Drive. Es bastante famoso y se utiliza en el entorno profesional también.  Da unos resultados muy buenos y cuenta con aplicación para Windows, Mac, iOS y Android.

## Instalación
Para instalarlo, otras veces nos ha bastado con bajarnos el paquete de su web y colocarlo en la raíz del servidor web, pero no sabemos por qué, está fallando este método. Así que toca instalarlo por ``apt-get``.

Bajamos y añadimos la clave del repositorio:
```bash
sudo wget -nv https://download.owncloud.org/download/repositories/production/Debian_9.0/Release.key -O Release.key
sudo apt-key add - < Release.key
```
Añadimos el repositorio al fichero ``sources.list``. Después, actualizamos e instalamos el paquete ``owncloud-files``.

```bash
echo 'deb http://download.owncloud.org/download/repositories/production/Debian_9.0/ /' > /etc/apt/sources.list.d/owncloud.list
apt-get update
apt-get install owncloud-files
```

En este punto, tendremos el directorio ``owncloud`` en ``/var/www/``. Sólo queda mover el contenido a la raíz del servidor web, o bien cambiar la raíz del servidor web a ``/var/www/owncloud/``.

### Paquetes adicionales necesarios
Antes de seguir con la instalación, tenemos que suplir las dependencias de paquetes. El nuevo ownCloud está preparado para trabajar con PHP7, la nueva versión de PHP. Es más eficiente, así que hemos decidido usarla. Antes de nada, Debian 9 **ya trae PHP7 por defecto**, pero no todos los módulos. Entonces, nosotros sólo hemos tenido que instalar los módulos necesarios:

```bash
apt-get install php7.0-zip php-xml php7.0-intl php7.0-xsl php-mbstring php7.0-gd  php7.0-curl php7-mysql php-xml
```

Si no hay ningún error, al entrar a la interfaz web de esta máquina, debería mostrarse el index para instalarlo:
![](https://imgur.com/bPcRw3s.png)

Como se puede observar, la dirección del servidor de BD al que apuntamos es la de la máquina que hemos configurado previamente. **La contraseña del usuario de la base de datos hay que introducirla hasheada**.

Tras esperar un tiempo (en la RPi 3 han sido diez minutos aproximadamente, demasiado largo), acabará la instalación. 

Pero, aunque la instalación ya esté acabada, al entrar como administrador, nos dará unas recomendaciones de seguridad, que son: 

 1. Mover el directorio raíz fuera de ``/var/www/``.
 2. Configurar el sistema de caché en el sistema, no en la BD.
 3. Activar el HTTPS.
 4. (Puede no salir) Corregir integridad de ficheros.

##### 1. Mover el directorio raíz fuera de ``/var/www``
Hemos optado por ponerlo en ``/var/opt/owncloud-data``.
```bash
mv /var/www/data/* /var/www/data/.* /var/opt/owncloud-data
chmod -R www-data:www-data /var/opt/owncloud-data
chmod -R o-rw /var/opt/owncloud-data
```
Como nos dio el consejo **después** de instalarse, tuvimos que modificar el archivo de configuración, ubicado en ``/var/www/config/config.php``, de esta manera:

```
...
'datadirectory' => '/var/opt/owncloud-data',
...
```

#####  2. Configurar el sistema de caché en el sistema, no en la BD
Primero, ponemos ownCloud en modo mantenimiento. Editamos el fichero de configuración, y añadimos:

```
...
'maintenance' => 'true',
...
```
Volvemos a la máquina de la BD, y entramos en el sistema de base de datos:
```bash
ssh usuario@bdserver
mariadb -u root -p
```

Dentro de la BD, borramos la tabla ``oc_file_locks``:
```sql
USE owncloud;
DELETE FROM oc_file_locks WHERE 1;
```

Cerramos la conexión ssh, y pasamos a instalar y configurar el servidor de caché (Redis). Hay varios métodos, otro es ACPu, pero el recomendado por el equipo de ownCloud es Redis.

```bash
apt-get install redis-server php-redis
```
Editamos el fichero de configuración ``/etc/redis/redis.conf``, y cambiamos el parámetro ``appendonly no`` a ``appendonly yes``. Reiniciamos Redis.

```bash
/etc/init.d/redis-server restart
```
Ejecutamos ``sysctl vm.overcommit_memory=1`` y añadimos la misma línea al final del fichero ``/etc/sysctl.conf``.

Ahora, vuelta al fichero de configuración de ownCloud (``/var/www/config/config.php``), añadimos al principio:

```
'filelocking.enabled' => true,
'memcache.local' => '\OC\Memcache\Redis',
'memcache.locking' => '\OC\Memcache\Redis',
'redis' => [
        'host' => 'localhost',
        'port' => 6379,
],﻿
```
y borramos la línea de antes, que decía ``'maintenance' => 'true'``.

Reiniciamos Apache.

##### 3. Activar el SSL
Este es el paso más sencillo. Hay que hacer lo mismo que [aquí](#ssl-en-máquina-secundaria). Además, editar el fichero de configuración de ownCloud (``/var/www/config/config.php``) y, en el apartado ``'trusted_domains'``, añadir una nueva línea que representa el dominio desde el que vamos a acceder. En este caso, es toledoaguilera.ddns.net. Quedaría así:

```
'trusted_domains' =>
	array (
		'toledoaguilera.ddns.net',
		'localhost',
	),
```

##### 4. (Opcional) Integridad de los ficheros
Esto puede ser que no sea necesario en otro caso, pero a nosotros nos ha salido. Se trata, básicamente, de ver qué ficheros están dando problemas y tomar las acciones pertinentes.  A nosotros nos decía que nos faltaba el ``.htaccess`` y el ``.user.ini``, así que hay que descargar [este paquete](https://download.owncloud.org/community/owncloud-10.0.8.zip), buscar esos ficheros, y colocarlos en el directorio ``/var/www/core``, que es donde estaba faltando.


# Conclusión
Ha sido un trabajo costoso y laborioso. Hemos tenido que coordinar cinco máquinas, instalarlas, configurarlas, probarlas... y, siendo sinceros, las Raspberry Pi, aunque se han portado bien, se nota que son ordenadores de 35€. Sin ir más  lejos, a ownCloud le cuesta cargar, y eso que hemos utilizado tarjetas de clase 10.

De todas formas, es un gustazo cuando accedes por enésima vez a la URL y funciona todo a la perfección. Es glorioso ver que tu trabajo funciona correctamente.
