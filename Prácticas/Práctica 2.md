#PRÁCTICA 2
    
***

##**CLONAR LA INFORMACIÓN DE UN SITIO WEB**


***

En esta segunda práctica se han alcanzado los siguientes objetivos:

- Aprender a copiar archivos mediante ssh
- Clonar contenido entre máquinas
- configurar el **ssh** para acceder a máquinas remotas sin contraseña
- Establecer tareas en **cron**

Para estos objetivos, hemos instalado la herramienta **rsync**

## Copia de archivos mediante **ssh**:

Mediante el comando:
  
--> "*tar czf - directorio | ssh equipodestino 'cat > ~/tar.tgz'*"
 
 
![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/copiassh.PNG)

## Clonación contenido entre máqiuinas

En primer lugar instalamos la herramienta rsync que nos ayudará a clonar cualquier carpeta desde una maquina en otra.
En nuestro caso, hemos clonado la carpeta */var/www/* mediante el comando:

--> "*rsync -avz -e ssh 192.168.1.10:/var/www/ /var/www/*"

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/clonrsync.PNG)

## Configuración de **ssh** para acceder sin contraseña 

Para acceder remotamente a una maquina mediante ssh, hasta ahora, se nos ha solicitado la contraseña de la máquina a la que queremos acceder. Esto nos es un problema a la hora de programar clonaciones ya que tendríamos que estar introduciendo la contraseña siempre para que se pudiera llevar a cabo la copia.

Mediante ssh-keygen generamos la clave pública-privada:

--> *ssh-keygen -b 4096 -t rsa*

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/key.PNG)

Luego, copiamos la clave pública al equipo remoto, añadiéndola al fichero *~/.ssh/know_hosts*:

--> *chmod 600 ~/.ssh/know_hosts*

Por último copiamos la clave mediante el comando:

--> -*ssh-copy-id 192.168.1.11*

A partir de ahora ya no nos pide la clave de acceso.

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/conexsincon.PNG)

##Establecer tareas con cron

Modificando correctamente el archivo /etc/crontab podemos decirle al servidor que realice copias del fichero /var/www/ de la maquina principal en la maquina remota cada hora. 

La sintaxis correcta quedaría así:

![img](https://github.com/JuanDiegoJr7/SWAP/blob/master/Pr%C3%A1cticas/Imagenes/cron.PNG)

***



