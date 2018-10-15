# Práctica 5
##### Alejandro Manuel Hernández Recio y Francisco Miguel Toledo Aguilera
https://www.github.com/alexhzr
https://www.github.com/maikeltoledo

## Escenario

 - Host: Windows 10 
 - Máquinas virtuales:
	 -  **P5M1: Servidor de BD I** 
		 - VBox Interface: 10.0.2.5 /24
	 - **P5M2: Servidor de BD II**
		 - VBox Interface: 10.0.2.6 /24


## Desarrollo 
### Crear una BD e insertar datos
Para crear la base de datos hay que entrar en el shell mysql con el comando ``mysql -u <usuario> -p``. A continuación, mediante sentencias SQL, se crea la BD y se inserta algún dato.

```sql
CREATE DATABASE <contactos>;
USE <contactos>;
CREATE TABLE datos(nombre VARCHAR(100), tlf INT);
INSERT INTO datos(nombre, tlf) VALUES ("Juanca Alaminos", 999999999);
```

![](https://imgur.com/8LwjmPW.png)

### Replicar una BD con mysqldump
``mysqldump`` es una herramienta que permite hacer volcados de las bases de datos que tengamos. Para hacer una copia de seguridad de la BD, es necesario decirle al servidor que no se acceda a la BD para cambiar datos.

```sql 
mysql -u root -p
FLUSH TABLES WITH READ LOCK;
quit;
```

Salimos de mysql y ejecutamos:

``mysqldump <nombre_db> -u root -p > /tmp/<nombre_db>.sql``

Quitamos el lock de las tablas:

```sql 
mysql -u root -p
UNLOCK TABLES;
quit;
```

![](https://imgur.com/hErAOJ5.png)

Y copiamos desde la máquina esclavo:

``scp P5M1:/tmp/<nombre_db>.sql /tmp/``

Creamos la BD y ejecutamos el script SQL que hemos copiado:

```sql
CREATE DATABASE <contactos>;
quit;
```

``mysql -u root -p contactos < /tmp/contactos.sql``

![](https://imgur.com/5qjZzfl.png)

### Replicación de BD maestro-esclavo
Para configurar el M-E primero configuramos el maestro. En el fichero de configuración ``/etc/mysql/mysql.conf.d/mysqld.cnf`` y modificamos las líneas:
```
# Esta de abajo se modifica comentándola, es decir, añadiendo '#'
#bind-address 127.0.0.1
log_error = /var/log/mysql/error.log
server-id = 1
log_bin = /var/log/mysql/bin.log
```

Y reiniciamos el servicio ``/etc/init.d/mysql restart``

Ahora vamos a la máquina esclavo. Tocamos el mismo fichero de antes, pero esta vez, ``server-id = 2``. Reiniciamos el servicio.

Desde el maestro creamos el usuario esclavo usado para la replicación:

```sql
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%'
IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;
```

Con ``SHOW MASTER STATUS`` obtenemos el ``File`` y ``Position`` que usaremos en la configuración del esclavo.

![](https://imgur.com/zzz9Vxb.png)

Volvemos al esclavo y en la shell de mysql escribimos el siguiente comando (se puede separar en varias líneas pulsando enter):

```sql
mysql> CHANGE MASTER TO MASTER_HOST='10.0.2.5',
-> MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
-> MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=980,
-> MASTER_PORT=3306;

# Arrancamos el esclavo
mysql> START SLAVE;
```

Aquí tuvimos un fallo y es que la máquina viene clonada de una máquina principal que hicimos al principio, por lo que el UUID del server de MySQL es idéntico en ambas.

![](https://imgur.com/ULgdnYe.png)

La solución es muy sencilla: basta con borrar el fichero ``/var/lib/mysql/auto.cnf`` y reiniciar el servicio.

Aquí se puede ver la replicación:

![](https://imgur.com/Kz11zlM.png)

### Configuración de replicación Maestro-Maestro
Para lograr la replicación en ambos sentidos, es necesario hacer lo mismo que hemos hecho en la máquina 1, ahora en la máquina 2.

Ejecutamos desde la shell del actual esclavo:

```sql
mysql> CREATE USER esclavo2 IDENTIFIED BY 'esclavo2';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo2'@'%'
IDENTIFIED BY 'esclavo2';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;
```

![](https://imgur.com/rQ6cAtg.png)

Nos quedamos con el valor del ``File`` y ``Position``:

![](https://imgur.com/GcVyk51.png)

Ahora en la máquina 1, ejecutamos:

```sql
mysql> CHANGE MASTER TO MASTER_HOST='10.0.2.6',
-> MASTER_USER='esclavo2', MASTER_PASSWORD='esclavo2',
-> MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=154,
-> MASTER_PORT=3306;
mysql> START SLAVE;
```
Y la prueba de que la replicación funciona en ambos sentidos:

![](https://imgur.com/V6EQ0Ua.png)
