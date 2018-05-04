![Imgur](https://i.imgur.com/9Hh6pP2.png)
# Autores
##### Francisco Miguel Toledo Aguilera y Alejandro Manuel Hérnandez Recio
# Introduction
En este proyecto vamos a realizar una granja web de altas prestaciones con raspberrys pi.

## Escenario

 * 5 raspberrys pi
 * 1 Switch
 * 1 dominio con No-ip (toledoaguilera.ddns.net)

## Topología
![Imgur](https://i.imgur.com/pGehuR1.jpg)
![Imgur](https://i.imgur.com/jnfl92y.jpg)

# Apertura de puertos y configuración de No-ip
### Cliente No-ip
El cliente de No-ip, se puede configurar de varias formas, una de ellas es instalando un proceso en cualquiera de los servidores para que actualice cada X tiempo la información sobre nuestra IP pública. En nuestro caso, hemos decidido utilizar el apartado de configuracíon de No-ip en el router, donde hay que definir el dominio e introducir nuestro usuario y contraseña de No-ip:
![Imgur](https://i.imgur.com/PiYf24F.jpg)

### Puertos
Necesitaremos abrir el puerto de SSH para administrar la granja desde fuera, este puerto apuntará al balanceador, desde el cual, podremos configurar los demás servidores.
Puerto HTTPS hacia el balanceador, en nuestro caso, sólo vamos a permitir peticiones HTTPS.
Puerto 9090 para la administración y gestion de Owncloud.
![Imgur](https://i.imgur.com/p78RnGU.jpg)
# Instalación y configuración de Apaches
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
La instalación del certificado primero se hace sobre la máquina principal (Apache 192.168.1.11), instalamos git:
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
Dentroo del directorio:
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
/etc/letsencrypt/live/toledoaguilera.ddne.net/
```
#### Sitio toledoaguilera.ddns.net
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
Ya tendríamos el certificado instalado en nuestra máquina principal.
#### SSL en máquina secundaria
Copiamos los siguientes directorios de la máquina principal a la secundaria:
```bash
/etc/letsencrypt/ && /usr/local/letsencryp/
```
En este punto seguimos los pasos del punto anterior:
##### Sitio toledoaguilera.ddns.net
Para añadir el nuevo sitio en `sites-available` y cambiar el fichero de hosts.
Ya tendríamos SSL en las dos máquinas.
# Instalación y Configuración de Nginx
Pasamos a configurar Balanceador(192.168.1.10), ejecutamos los siguientes comandos:
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
En este punto ya tendríamos configurado nuestros dos servidores webs apache junto con el balanceador de carga nginx.
