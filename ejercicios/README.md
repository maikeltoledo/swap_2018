# Ejercicios de clase

Ejercicios de cada uno de los temas de teoría vistos en clase.

## Tema 2: Alta disponibiliad y escalabilidad
**Ejercicio T2.1**: Calcular la disponibilidad del sistema descrito en edgeblog.net si en cada subsistema tenemos en total 3 elementos.

| Componet     | Dos replicas |  Calculos nuevos                         | Tres replicas |
|--------------| ------------ | ---------------------------------------- | ------------- |
| Web          | 97,75%       | 97,75% + (1 – 97,75%) * 97,75%           | 99,949375%    |
| Application  | 99%          | 99% + (1 – 99%) * 99%                    | 99,99%        |
| Database     | 99,9999%     | 99,9999% + (1 – 99,9999%) * 99,9999%     | 100%          |
| DNS          | 99,96%       | 99,96% + (1 – 99,96%) * 99,96%           | 99,999984%    |
| Firewall     | 97,75%       | 97,75% + (1 – 97,75%) * 97,75%           | 99,949375%    |
| Switch       | 99,99%       | 99,99% + (1 – 99,99%) * 99,99%           | 99,999999%    |
| Data Center  | 99,99%       | 99,99% + (1 – 99,99%) * 99,99%           | 99,999999%    |
| ISP          | 97,75%       | 99,75% + (1 – 99,75%) * 99,75%           | 99,999375%    |


_Total_ = 99,949375% \* 99,99% \* 100% \* 99,999984% \* 99,949375% \* 99,999999% \*
99,999999% \* 99,999375% = 99,8881435%

**Ejercicio T2.2**: Buscar frameworks y librerías para diferentes lenguajes que permitan hacer aplicaciones altamente disponibles con relativa facilidad. Como ejemplo, examina PM2: https://github.com/Unitech/pm2 que sirve para administrar clústeres de NodeJS.
<![endif]-->

[**Angular.js**](https://angularjs.org/): Un framework basado en _JavaScript_

[**react**](https://facebook.github.io/react/): Liberado por Facebook, en _JavaScript_, permite desarrollar aplicaciones móviles para IOS y Android

[**ionic**](http://ionicframework.com/): Para móviles, usando HTML, Js, Sass y Angular

[**Meteor**](https://www.meteor.com/): En _JavaScript_, para web y móviles

[**Ruby on Rails**](http://www.linuxlinks.com/article/20120525000539219/RubyonRails.html): Framework MVC basado en Ruby, orientado al desarrollo de aplicaciones web

[**CodeIgniter**](http://www.linuxlinks.com/article/20120525000531497/CodeIgniter.html): Poderoso framework PHP liviano y rápido

[**Kohana**](http://kohanaframework.org/): Un fork de CodeIgniter, _Gracias a Samuel por mencionarlo en los comentarios._

[**Django**](http://www.linuxlinks.com/article/20120525000545879/Django.html): Framework Python que promueve el desarrollo rápido y el diseño limpio

[**CakePHP**](http://www.linuxlinks.com/article/20120525000252509/CakePHP.html): Framework MVC para PHP de desarrollo rápido

[**Zend Framework**](http://www.linuxlinks.com/article/20120525000536311/ZendFramework.html): Framework para PHP 5, simple, claro y open-source

[**Yii**](http://www.linuxlinks.com/article/2012052500054269/Yii.html): Framework PHP de alto rendimiento basado en componentes

[**Pylons**](http://www.linuxlinks.com/article/2012052500055227/Pylons.html): Framework web para Python que enfatiza la flexibilidad y el desarrollo rápido

[**Catalyst**](http://www.linuxlinks.com/article/20120525000602635/Catalyst.html): Framework para aplicaciones web MVC elegante

[**Symfony**](http://www.linuxlinks.com/article/20120525000534344/Symfony.html): Framework full-stack


- **PM2 Runtime**: es un administrador en tiempo de ejecución de servidores NodeJS. Entre sus funciones se encuentran la de gestión de procesos, reinicios y ejecución de script en el sistema, etc.


**Ejercicio T2.3**: ¿Cómo analizar el nivel de carga de cada uno de los subsistemas en el servidor? Buscar herramientas y aprender a usarlas.

He encontrado las siguientes:

- Jmeter

- apache benchmark

- Selenium
<![endif]-->

**Ejercicio T2.4**: Buscar diferentes tipos de productos:

- Buscar ejemplos de balanceadores software y hardware (productos comerciales).

	- ManageEngine OpManager

	- Incapsula

	- Loadbalancer.org

	- Load Balancer ADC

	- Barracuda Load Balancer ADC

- Buscar productos comerciales para servidores de aplicaciones.

	- Google Web Server

	- Microsoft IIS

	- Webempresa.com

- Buscar productos comerciales para servidores de almacenamiento.

	- ovh.es

	- DropBox

	- GoogleDrive

	- OneDrive

	- iCloud
## Tema3: La red de una granja web
**Ejercicio T3.1**: Buscar con qué órdenes de terminal o herramientas gráficas podemos configurar bajo Windows y bajo Linux el enrutamiento del tráfico de un servidor para pasar el tráfico desde una subred a otra.

-   **Windows:** route ADD "IP RED" MASK "MASCARA" “PUERTA DE ENLACE”
-   **Linux:** route add default gw "PUERTA DE ENLACE" route add -net "SEGUNDA SUBRED" netmask "MASCARA" gw "PUERTA DE ENLACE 2º SUBRED"

**Ejercicio T3.2:** Buscar con qué órdenes de terminal o herramientas gráficas podemos configurar bajo Windows y bajo Linux el filtrado y bloqueo de paquetes.

- **Windows 10:** Buscamos Firewall de Windows defender y nos llevará automáticamente al menú principal del cortafuegos. Si pinchamos en configuración avanzada podemos crear reglas de entrada / salida a nuestro server.

- **Linux:** se hace con iptables, en la actualidad, se usa también el entorno grafico GUFW, que da mayor facilidad a la hora de definir reglas.
<![endif]-->

## TEMA 4: Balanceo de carga

**Ejercicio T4.1**: Buscar información sobre cuánto costaría en la actualidad un mainframe. Comparar precio y potencia entre esa máquina y una granja web de unas prestaciones similares.

Tras investigar, veo que IBM ha recortado los precios de sus mainframes en los últimos años.

Por ejemplo, el precio de IBM zEnterprise 114 es de 410 mil dólares.

**Ejercicio T4.2**: Buscar información sobre precio y características de balanceadores hardware específicos. Compara las prestaciones que ofrecen unos y otros.

|      | Cisco ACE load balancing ACE30 |  LoadMaster LM-8020M | 
|--------------| ------------ | -------------------------------- |
| Precio          | $1246.00       | 60.000€        |
| Peso (Kg)  | 12          | 17               |
| RAM     | 16GB     | 8GB    |
| Conexiones |Ethernet, Fast Ethernet, Gigabit Ethernet       | Ethernet, Fast Ethernet, Gigabit Ethernet          |
| Método de auten.| New connections (SSL) | SSL TPS (2K Keys)   |

**Ejercicio T4.3**: Buscar información sobre los métodos de balanceo que implementan los dispositivos recogidos en el ejercicio 4.2

Los métodos de balanceo usados por LM-8020M son:

- SDN Adaptive

- Round Robin

- Weighted Round Robin

- Least Connection

- Weighted Least Connection

- Agent‐based Adaptive

- Chained Failover (Fixed Weighting)

- Source‐IP Hash

- Layer 7 Content Switching

- Global Server Load Balancing (GSLB)

- AD Group based traffic steering


**Ejercicio T4.4**: Instala y configura en una máquina virtual el balanceador ZenLoadBalancer. Compara con la dificultad de la instalación y configuración usando nginx o haproxy (práctica 3).

Personalmente no lo he probado, pero dejo un video de demostración donde se puede comprobar que la configuración de ZenLoadBalancer es más fácil que haproxy por su interfaz web.

[Enlace a YouTube](https://www.youtube.com/watch?v=8pnK_DlY-vM)

**Ejercicio T4.5:** Probar las diferentes maneras de redirección HTTP. ¿Cuál es adecuada y cuál no lo es para hacer balanceo de carga global? ¿Por qué?

Tenemos dos tipos de redireccionamiento:

- Redireccionamiento 301: Se puede definir este tipo de redireccion como “PERMANENTE”. Esto indica que todo contenido de una URL antigua se mueva de forma permanente a la URL nueva. De esta manera los buscadores dejarán de tener en cuenta la antigua URL y le traspasarán “aproximadamente” el 90% de la fuerza SEO a la nueva Web.

- Redireccionamiento 302: Se puede definir este tipo de redirección como “TEMPORAL”. Esto indica que todo el contenido de una URL antigua se mueva de forma temporal a la URL nueva. De esta manera redirigimos a los buscadores y usuarios de la antigua URL a la nueva pero no traspasamos la fuerza del SEO de una a otra.

**Ejercicio T4.6**: Buscar información sobre los bloques de IP para los distintos países o continentes. Implementar en JavaScript o PHP la detección de la zona desde donde se conecta un usuario.

Extracto sacado de monografías.com:

Existen 4 organizaciones continentales que tienen la misión de asignar y controlar el uso de direcciones IP por parte de los países. Estas son APNIC (www.apnic.net) para Asia y el Pacífico, ARIN (www.arin.net) para Norteamérica, LACNIC (www.lacnic.net) para Latinoamérica y el Caribe, y RIPE (www.ripe.net) para Europa, Africa del norte y Rusia).

Cada una de estas organizaciones mantiene un registro detallado de los grupos o bloques de direcciones IP que se han asignado a los distintos países. Si juntamos la información de estas cuatro entidades, podemos construir una gran base de datos conteniendo todos los bloques de direcciones IP del mundo asignados a sus respectivos países.

![Imgur](https://i.imgur.com/31lWMpZ.jpg)

**Ejercicio T4.7:** Buscar información sobre métodos y herramientas para implementar GSLB.

Las herramientas y métodos más utilizados para implementar GSLB son:

1.  Round Robin
2.  Least Connections
3.  Least Response Time
4.  Least Bandwidth
5.  Least Packets
6.  Source IP Hash
7.  Custom Load
8.  Round Trip Time (RTT)
9.  NetScaler (Herramienta)

## TEMA 5: Medir prestaciones

**Ejercicio 5.1**: Buscar información sobre cómo calcular el número de conexiones por segundo.

Para monitorizar el número de conexiones por segundo, debemos de activar la opción de mod-status. Podemos ver la opción para activarla en el siguiente código:
```
<Location /server-status>

	SetHandler server-status
	Order Deny,Allow
	Deny from all
	Allow from .website.com

</Location>
```
**Ejercicio 5.2**: Instalar wireshark y observar cómo fluye el tráfico de red en uno de los servidores web mientras se le hacen peticiones HTTP.

![Imgur](https://i.imgur.com/SsZATxZ.jpg)

<![endif]-->

**Ejercicio 5.3**: Buscar información sobre herramientas para monitorizar las prestaciones de un servidor.

- Nagios: sistema de código abierto utilizado para la monitorización de redes. Monitoriza los servicios de red como el correo, web, etc. también monitoriza los recursos de sistemas hardware como el procesador, uso de los discos, memoria, etc.

- Cacti: casi todo se configura a través de su interfaz web. No es menos fiable, la compatibilidad con SNMP es un gran fallo.

- Munin: Es una herramienta multiplataforma basada en web, utilizada en el monitoreo de los recursos en red. Está diseñada para ser plug and play. Su arquitectura es bastante sencilla: un servidor que centraliza los datos enviados por los agentes instalados en cada cliente. Permite monitorizar muchos parámetros y visualizarlos en cómodas gráficas.

- Zabbix: es un sistema para monitorear la capacidad, el rendimiento y la disponibilidad de los servidores, equipos, aplicaciones y bases de datos. Además ofrece características avanzadas de monitoreo, alertas y visualización, que incluso, algunas de las mejores aplicaciones comerciales de este tipo no ofrecen.
<![endif]-->

## TEMA 6: Asegurar el sistema web

**Ejercicio T6.1**: Aplicar con iptables una política de denegar todo el tráfico en una de las máquinas de prácticas. Comprobar el funcionamiento.
```
sudo iptables -A INPUT -j DROP
listamos las reglas:
sudo iptables
```

![Imgur](https://i.imgur.com/6cX9e5d.jpg)

**Ejercicio T6.2**: Comprobar qué puertos tienen abiertos nuestras máquinas, su estado, y qué programa o demonio lo ocupa.

Usamos el comando **netstat –a** en nuestro servidor:

![Imgur](https://i.imgur.com/nKKSKtL.jpg)

- **Ataque DoS:** En un ataque de denegación de servicio (DoS), un atacante intenta evitar la legitimidad de que los usuarios accedan a información o a los servicios. El tipo más común y obvio de ataque DoS ocurre cuando un atacante "inunda" una red con información.

- **Ping Flood:** se basa en enviar a la víctima una cantidad abrumadora de paquetes ping, usualmente usando el comando "ping" de UNIX como hosts. Es muy simple de lanzar, el requisito principal es tener acceso a un ancho de banda mayor que la víctima.

- **Ping de la muerte:** El atacante envía un paquete ICMP de más de 65.536 bytes. Como el sistema operativo no sabe cómo manejar un paquete tan grande, se congela o se cuelga en el momento de volver a montarlo. Hoy en día, el sistema operativo descarta dichos paquetes por sí mismo. Pero debía enumerar dicho ataque mítico.

- **ARP Spoofing:** puede permitir que un atacante detecte frameworks de datos en una red de área local (LAN), modifique el tráfico o detenga el tráfico por completo. El ataque solo se puede usar en redes que realmente usan ARP y no en otro método de resolución de direcciones.

- **ACK flood:** Esta es una técnica para enviar un paquete TCP / ACK al objetivo a menudo con una dirección IP falsificada. Es muy similar a los ataques de inundación TCP / SYN.

- **Ataque FTP Bounce:** El atacante puede conectarse a los servidores FTP y tiene la intención de enviar archivos a otros usuarios / máquinas que usan el comando PORT. Para que el servidor FTP intente enviar el archivo a otras máquinas en un puerto específico y verifique que el puerto esté abierto. Es obvio que la transferencia de FTP estaría permitida en los firewalls. En estos días casi todo los servidores FTP se implementan con el comando PORT desactivado.

- **TCP Session Hijacking:** Es el caso cuando el "Hacker" toma la sesión TCP existente, ya establecido entre las dos partes. En la mayoría de las sesión TCP la autenticación ocurre al comienzo de la sesión, los piratas informáticos realizan este ataque en dicho momento.
Fuente: [Enlace](https://ciberseguridad.blog/25-tipos-de-ataques-informaticos-y-como-prevenirlos/)
