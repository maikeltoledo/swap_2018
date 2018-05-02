# Práctica 4
##### Alejandro Manuel Hernández Recio y Francisco Miguel Toledo Aguilera

## Escenario

 - Host: Windows 10 
 - Máquinas virtuales:
	 -  **P4M1: Apache.** 
		 - VBox Interface: 10.0.2.4 /24
		 - Adaptador puente: 192.168.1.64 /24
	 - **P4M2: Apache.**
		 - VBox Interface: 10.0.2.5 /24
	 - **P4Balanceador: Balanceador Nginx.**
		 - VBox Interface: 10.0.2.3 /24

## Desarrollo 
### Instalar certificado SSL autofirmado para configurar acceso por HTTPS

El certificado autofirmado garantiza que el tráfico viaje cifrado, pero a ojos de un navegador, el certificado podría ser no válido porque es autofirmado. Nadie nos asegura que quien haya firmado el certificado sea quien dice ser.

#### Generar e instalar un certificado autofirmado
 Primero tenemos que activar el módulo SSL de Apache. Con privilegios, ejecutamos:
 ``a2enmod ssl``
 
 Y reiniciamos el servidor. A continuación, creamos un directorio para almacenar los certificados (en este caso usaremos ``/etc/apache/ssl``) y el certificado en sí.
 
```shell
mkdir /etc/apache2/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```
Nos va a pedir una serie de parámetros para configurar el certificado. Es ir rellenándolos con los datos que queramos y ya está.

![Configurar certificado](https://i.imgur.com/mZl2dv6.png "Configurar certificado")
 
 Editamos el archivo de configuración del sitio default-ssl (``/etc/apache2/sites-available/default-ssl``) y añadimos las líneas resaltadas:
 
 ![](https://imgur.com/BW68iwP.png)
 
 Activamos el sitio y reiniciamos apache:
 
![](https://imgur.com/ij8YheI.png)
 
 Aquí podemos comprobar que el acceso mediante HTTPS se ha hecho correctamente:
 ![](https://imgur.com/OLsZE8n.png)
 ### Configuración del cortafuegos
 Para configurar el cortafuegos en Linux vamos a usar ``iptables``. Hemos creado un script que cada vez que se ejecuta, limpia las reglas anteriores y vuelve a configurar las nuevas. El contenido de dicho script es el siguiente:
 
![](https://imgur.com/lCiTxgy.png)

Acceso funcionando por HTTP y HTTPS:
![](https://imgur.com/sj4N2Gw.png)

Hemos automatizado el script incluyéndolo en el fichero ``crontab``.

![](https://imgur.com/jmuFsmS.png)

### Configuración del balanceador
Para que el balanceador pueda dirigir el tráfico HTTPS hace falta que tenga el certificado configurado anteriormente. Para ello, lo hemos copiado con ``scp``.
![](https://imgur.com/EMrMHNo.png)

Una vez los tenemos, hay que configurar ``nginx``. Editamos el fichero del sitio y añadimos las líneas resaltadas.

![](https://imgur.com/yPDQhDU.png)

Y la prueba final. Accedemos al balanceador por HTTPS y nos redirecciona correctamente:

![](https://imgur.com/5U4by1F.png)