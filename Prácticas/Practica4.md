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


## Configuración del cortafuegos





























