# PRÁCTICA 4 #
   
***

## **ASEGURAR LA GRANJA WEB** #


***

En esta cuarta práctica cumplimos los siguientes objetivos sonbre nuestra granja web:

- Instalar un certificado SSL para configurar el acceso HTTPS a los servidores
- Configurar las reglas del cortafuegos para proteger la granja web.

## Instalar un certificado **SSL** autofirmado para configurar el acceso por HTTPS:

Un certificado *SSL* sirve para brindar seguridad al visitante de su página web, una manera de decirles a sus clientes que el sitio es auténtico, real y confiable para ingresar datos personales.

El protocolo **SSL** (Secure Sockets Layer) es un protocolo de comunicación que se ubica en la pila de protocolos sobre **TCP/IP**.

SSL proporciona servicios de comunicación segura entre *cliente* y *servidor*:

- Autenticación (usado certificados)
- Integridad (mediante firmas digitales)
- Privacidad (mediante incriptación)

Para obtener un certificado SSL e instalarlo en nuestro servidor, podemos seguir diferentes caminos:

- Mediante una autoridad de certificación
- Crear nuestros propios certificados SSL (con la herramienta ** *openssl* **).
- Utilizar certificados del proyecto *Certbot* (antes Let's Encrypt)

###Generar e instalar un certificado autofirmado

Para generar un certificado SSL autofirmado en Ubuntu Server solo debemos activar el módulo SSL de Apache, generar los certificados y especificarle la ruta a los certificados en la configuración.

        $ a2enmod ssl
        $ service apache2 restart
        $ mkdir /etc/apache2/ssl
        $ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key-out /etc/apache2/ssl/apache.crt

A continuación editamos el archivo de configuración del sitio default-ssl:

        nano /etc/apache2/sites-available/default-ssl

Y agregamos estas líneas debajo de donde pone SSLEngine on:

        SSLCertificateFile /etc/apache2/ssl/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/apache.key

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/4-defaultssl.PNG)

Activamos el sistio default-ssl y reiniciamos apache:

        a2ensite default-ssl
        service apache2 reload

Ahora traspasamos lo realizado en el servidor 1 al resto de servidores y al balanceador mediante la herramienta ** *rsync* **:

        rsync -avz -e ssh ubuntus@192.168.1.10:/etc/apache2/ssl /etc/apache2/ssl

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/4-traspaso.PNG)


Una vez traspasado todo, tenemos que **configurar nginx adeacuadamente para aceptar y balancear correctamente tanto el tráfico HTTP como el HTTPS**:

Para ello añadimos al archivo /etc/nginx/conf.d/default.conf:

        listen 443 ssl;
        ssl_certificate /etc/apache2/ssl/apache.crt;
        ssl_certificate_key /etc/apache2/ssl/apache.key;

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/4-sslpermision.PNG)


Para hacer las peticiones por HTTPS, ejecutaremos:

        curl –k https://ipmaquina1/index.html

En la imagen observamos como funciona tanto con *http* como *https*:

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/4-finalssl.PNG)


## Configuración del cortafuegos:

Un cortafuegos es un componente esencial que protege la granja web de accesos indebidos. Son dispositivos colocados entre subredes para realizar diferentes tareas de manejo de paquetes. Actúa como el guardián de la puerta al sistema web, permitiendo el tráfico autorizado y denegando el resto.

Todos los paquetes **TCP/IP** que entran o salen de nuestra granja web tienen que pasar por el cortafuegos, que los examina y bloquea aquellos que no cumplen los criterios de seguridad que establecemos. Estos criterios se configuran mediante un conjunto de reglas:

- Rango de puertos
- Direcciones IP
- Rangos de IP
- Tráfico TCP o UDP
- Etc

###Configuración del cortafuegos **iptables** en Linux:

####¿Qué es *iptables*?

iptables es una herramienta de cortafuegos, de espacio de usuario, con la que el superusuario define reglas de filtrado de paquetes, de traducción de direcciones de red y mantiene registros de log. Esta herramienta está construida sobre *Netfilter*, una parte del núcleo de Linux que permite interceptar y manipular paquetes.

Vamos a establecer una lista de reglas con las que definir qué acciones hacer con cada paquete en función de la información que incluye. 

Para la configuración adecuada de iptables, hemos establecido en primer lugar y como reglas por defecto la denegación de todo el tráfico, salvo el que permitamos después explícitamente. 

###Uso de la aplicación **iptables**:

Para facilitar el uso de esta herramienta, creamos un script que almacene las ordenes que queremos que se ejecuten siempre y solo nos quedaría decirle al servidor que lo ejecute al iniciarse:

        #Eliminar cualquier orden anterior
        iptables -F
        iptables -X


        #Establecer política por defecto, denegar cualquier servicio
        iptables -P INPUT DROP
        iptables -P OUTPUT DROP
        iptables -P FORWARD DRÒP

        #Permitir cualquier acceso desde localhost (interface lo):
        iptables -A INPUT -i lo -j ACCEPT
        iptables -A OUTPUT -o lo -j ACCEPT
        # Abrir el puerto 22 para permitir el acceso por SSH
        iptables -A INPUT -p tcp --dport 22 -j ACCEPT
        iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
        # Abrir los puertos HTTP (80) de servidor web
        iptables -A INPUT -p tcp --dport 80 -j ACCEPT
        iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
        # Abrir los puertos HTTPS (443) de servidor web
        iptables -A INPUT -p tcp --dport 443 -j ACCEPT
        iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT

Una vez realizado, lo comprobamos y debería darnos acceso a la maquina servidora:

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/4-scriptprueba.PNG)

A continuación nos queda decirle al sistema que lo ejecute cada vez que se inicie. Lo haremos mediante el comando:

        $ sudo mv /home/ubuntus/scriptiptables.sh /etc/init.d/
        $ sudo update-rc.d scriptiptables.sh defaults

Y vemos como al reiniciar, el cortafuegos se ejecuta automáticamente:

![img]()
















