# PRÁCTICA 3 #
   
***

## **BALANCEO DE CARGA** #


***

En esta tercera práctica se han alcanzado los siguientes objetivos:

- Configurar una red entre varias máquinas de forma que tengamos un balanceador que reparta la carga.
- Solucionar un problema: "la **sobrecarga** de servidores"

El software de balanceo que se ha utilizado es: 

- nginx --> como proxy
- haproxy

Para todo ello, se ha creado una tercera máquina que hará de balanceador mediante el software antes mencionados, y se usarán los 2 servidores de las anteriores prácticas.

## Balanceador con **nginx**:

En primer lugar, comenzaremos por instalar el software, mediate el comando:

    sudo apt-get install nginx 

A continuación iniciamos el software:

    sudo systemctl start nginx

La configuración básica de nginx no nos vale tal cual, así que realizamos los siguientes cambios en el archivo "*/etc/nginx/conf.d/default.conf*" de manera que quede así:

    upstream apaches {
        server 172.16.168.130;
        server 172.16.168.131;
    }

    server{
        listen 80;
        server_name balanceador;
        access_log /var/log/nginx/balanceador.access.log;
        error_log /var/log/nginx/balanceador.error.log;
        root /var/www/;
        
        location /
        {
        proxy_pass http://apaches;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        }
    }

Una vez creado el archivo, activamos de nuevo nginx y desde una maquina cualquiera (en nuestro ejemplo usamos uno de los apaches), hacemos un curl a la ip del balanceador y vemos como va reenviandonos al servidor 1 y 2 alternativamente, puesto que tenemos asignado el mismo peso a ambos servidores. 

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/pruebanginxbasic.PNG)

También podemos asignarle, por ejemplo, pesos a los servidores para que reciban distinta carga (Supondremos que el servidor 1 es menos potente y queremos que el 2 reciba por lo tanto más carga):

    uptream apaches {
        server 192.168.1.10 weight=1;
        server 192.168.1.11 weight=2;
    }

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/pruebanginxpesos.PNG)

Otras opciones que podemos usar son las siguientes:

- Si queremos que una vez iniciada una sesión desde una ip se manenga el mismo servidor con ella, añadimos "*ip_hash*" al principio del upstream:
    
![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/nginxip_hash.PNG)

- Podemos hacer que un servidor aparezca offline para hacer reparaciones añadiendo *down* a un server (*server 192.168.1.10 down;*)


## Balanceador con **haproxy**:

haproxy es un balanceador de carga y también proxy, de forma que puede balancear cualquier tipo de tráfico. 

Para instalarlo, basta con utilizar el apt-get install:

     sudo apt-get install haproxy

Como la configuración que trae por defecto no nos vale, usaremos la siguiente (modificación del archivo /etc/haproxy/haproxy.cfg):

    global
        daemon
        maxconn 256

    defaults
        mode http
        contimeout 4000
        clitimeout 42000
        srvtimeout 43000

    frontend http-in
        bind *:80
        default_backend servers

    backend servers
        server m1 172.16.168.130:80 maxconn 32
        server m2 172.16.168.131:80 maxconn 32    

Una vez modificada esta configuración, lanzamos el servicio haproxy mediante la orden:

    sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg

Una vez iniciado, comprobamos que funciona igual que con haproxy:

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/haproxybasico.PNG)



## Someter a una **alta carga** el servidor balanceado:

Para este apartado utilizamos una máquina externa a nuestra granja web, a la que hemos llamado "clientes". En ella ejecutamos la herramienta *Apache Benchmark* como sigue:

    ab -n 1000 -c 10 http://192.168.1.15/index.html

Usando **nginx** como balanceador obtenemos los siguientes resultados:

![img]()

Usando **haproxy** como balanceador obtenemos los siguientes resultados:

![img]()




***
