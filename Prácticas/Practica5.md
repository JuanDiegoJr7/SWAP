# PRÁCTICA 5 #
   
***

## **REPLICACIÓN DE BASES DE DATOS MySQL** #


***

En esta quita práctica cumplimos los siguientes objetivos sobre nuestra granja web:

- Copiar archivos de copia de seguridad mediante sh.
- Clonar manualmente BD entre máquinas.
- Configurar la estructura maestro-esclavo entre dos máquinas y realizar el clonado automático de la información.

## Crear un tar con ficheros locales y copiarlos en un equipo remoto.

Como ya sabemos de anteriores prácticas, para realizar esta copia de un tar de un servidor a otro, basta con ejecutar la orden:

        tar czf - directorio | ssh equipodestino 'cat > ~/tar.tgz'

Que en nuestro quitrcaso quedaría así por ejemplo:

        tar czf - MiDirectorio | ssh 192.168.1.11 'cat > ~/tar.tgz'

Para sincronizar Bases de Datos, existen mejores herramientas dado que una sincronización suele manejar grandes cantidades de datos.

## Crear una BD e instertar datos.

Crearemos una base de datos MySQL mediante las siguientes órdenes:

        mysql -uroot -p
        Enter password: <TECLEAR LA CLAVE SI NO ES VACÍA>

        mysql> create database contactos;
            Query OK, 1 row affected (0,00 sec)

        mysql> use contactos;
            Database changed

        mysql> show tables;
            Empty set (0,00 sec)

        mysql> create table datos(nombre varchar(100),tlf int);
            Query OK, 0 rows affected (0,01 sec)

        mysql> show tables;
                +---------------------+
                | Tables_in_contactos |
                +---------------------+
                | datos |
                +---------------------+

                1 row in set (0,00 sec)

        mysql> insert into datos(nombre,tlf) values ("pepe",95834987);
            Query OK, 1 row affected (0,00 sec)
    
        mysql> select * from datos;
                +---------+-----------+
                | nombre | tlf |
                +---------+-----------+
                | pepe | 95834987 |
                +---------+-----------+
                    3 rows in set (0,00 sec)

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/5-CreateBD.PNG)

## Replicar una BD MySQL con ** *mysqldump* **

Anteriormente hablábamos de la existencia de unas herramientas para clonar Bases de Datos de manera eficiente al usar grandes cantidades de datos. Una de ellas es **mysqldump**.

La sintaxis de uso es:

        mysqldump ejemplodb -u root -p > /root/ejemplobd.sql


Esto puede ser suficiente, pero tenemos que tener en cuenta que los datos pueden estar actualizándose constantemente en el servidor de BD principal. En este caso, antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se
acceda a la BD para cambiar nada.

        mysql -u root –p

        mysql> FLUSH TABLES WITH READ LOCK;
        mysql> quit

Es ahora cuando podemos realizar el mysqldump sin riesgo.
Tras realizarlo es importante recordar que tenemos bloqueadas las tabalas, procederemos a desbloquearlas de la siguiente manera:

        mysql -u root -p
        
        mysql> UNLOCK TABLES;
        mysql> quit

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/5-sqldump1.PNG)

Una vez creado el archivo "contactos.sql", desde nuestro otro servido lo rescatamos:

        scp 192.168.1.10:/tmp/contactos.sql /tmp/


El archivo creado mediante *mysqldump* **NO CREA LAS SENTENCIAS DE CREACION DE LA BASE DE DATOS** por lo que debemos crear en un primer paso la base de datos nosotros mismos.

        Máquina 2:

        mysql -u root -p 
        
        msql> CREATE DATABASE 'contactos';
        msql> quit

Por último solo queda restaurar la copia de la otra máquina:

        mysql -u root -p contactos < /tmp/contactos.sql

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Im%C3%A1genes/5-copiacorrecta1.PNG)
Podemos ver en la imagen como aparece la tabla tal y como se creó en la Máquina 1.

Como nota final del apartado, recordar que también podríamos haber realizado la copia directamente usando un "pipe" a un ssh para exportar los datos al mismo tiempo:

        mysqldump contactos -u root -p | ssh 192.168.1.11 mysql

## Replicación de BD mediante una configuración maestro-esclavo.

Vamos a configurar el demonio para hacer la replicación de BD sobre un esclavo. Así, no tiene que haber una persona realizando la replicación a mano.

Primero editamos la configuración de mysql que se encuentra en /etc/mysql/mysql.conf.d/mysql.cnf:

        sudo nano /etc/mysql/mysql.conf.d/mysql.cnf

Comentamos el parámetro "bind-address" que sirve para que escuche a un servidor:

Y descomentamos las líneas:

        log_errror = /var/log/mysql/error.log
        server-id = 1
        log_bin = /var/log/mysql/bin.log
        
Una vez realizados los cambios anteriores, reseteamos el servicio:

        /etc/init.d/mysql restart

![img]()

A continuación pasamos a configurar la máquina 2 que actúa como esclavo:

La configuración es similar a la del maestro, con la diferencia de que el server-id en esta ocasión será 2.

Una vez más, si no da ningún error, habremos tenido éxito. Podemos volver al maestro para crear un usuario y darle permisos de acceso para la replicación. Entramos en mysql y ejecutamos las siguientes sentencias:

        mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
        mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
        mysql> FLUSH PRIVILEGES;
        mysql> FLUSH TABLES;
        mysql> FLUSH TABLES WITH READ LOCK;

Para finalizar con la configuración en el maestro, obtenemos los datos de la BD que vamos a replicar para posteriormente usarlos en la configuración del esclavo:

        mysql> SHOW MASTER STATUS;

![img] ()

Ahora, en la maquina 2 (el esclavo), iniciamos mysql e introducimos las siguientes ordenes para configurar el esclavo:

        mysql> CHANGE MASTER TO MASTER_HOST='192.168.31.200', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=501, MASTER_PORT=3306;

Y por último iniciamos el esclavo:

        mysql> START SLAVE;

![img]()

Para comprobar que todo funciona correctamente, ejecutamos en el esclavo la orden:

        mysql> SHOW SLAVE STATUS\G

El valor de "Seconds_Behind_Master" debe ser distinto de "null". Si no es así, ahí mismo se verán los errores. 

Ahora, cualquier cambio realizado en el maestro se verá reflejado también en el esclavo.

---





