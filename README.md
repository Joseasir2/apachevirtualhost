# 1.Archivos de configuración

Primero que todo crearemos los archivos de configuración que hacen falta para arrancar el apache. Se usan los siguientes comandos:


$ docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > httpd.conf

$ docker run --rm httpd:2.4 cat /usr/local/apache2/conf/mime.types > mime.types

Estos ficheros irán guardados en "conf".


# 2.Ficheros de conf con sus archivos y bases de datos en zonas

Una vez creados los ficheros de configuración necesitaremos los ficheros de "named.conf" en la carpeta de "configuración". OJO, no es la carpeta "conf". Necesitaremos los típicos archivos de "named.conf, named.conf.fedault-zones, named.conf.local y named.conf.options". También necesitaremos crear las bases de datos "db.fabulasoscuras.int" y "db.fabulasmaravillosas.int" en la carpeta "zonas". Debe ser así:

@		IN NS	ns.fabulasmaravillosas.int.
ns		IN A		33.33.5.14
test	IN A		33.33.5.34
www	IN CNAME	ns
texto	IN TXT		mensaje


@		IN NS	ns.fabulasoscuras.int.
ns		IN A		33.33.5.14
test	IN A		33.33.5.33
www	IN CNAME	ns
texto	IN TXT		mensaje



La ip debe estar apuntando al servidor apache, es decir tiene que tener la IP del apache.


# 3.Archivos html

Crearemos una carpeta llamada "páginas" y dentro archivos html, en mi caso he creado "holamundo.html" y "adiosmundo.html".


# 4.Docker-compose

Configuraremos el docker compose con 3 contenedores, 1 de apache, otro de dns y otro de cliente con sus IPs fijas tal que así:

services:
  servidor_apache:
    container_name: asir_apache_web
    image: httpd:latest
    ports:
      - "80:80"
    # Mapeamos el directorio raíz (equipo:contenedor)  
    volumes:
      - ./contenidoweb:/usr/local/apache2/htdocs
      - ./conf:/usr/local/apache2/conf   
    networks:
      red_33:
        ipv4_address: 33.33.5.14

  servidor_dns:
    container_name: asir_servidor_dns
    image: ubuntu/bind9
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      #Mapeo de puertos
    networks:
      red_33:
        ipv4_address: 33.33.5.1
    volumes:
      - ./configuracion:/etc/bind
      - ./zonas:/var/lib/bind
      #Para mapear los directorios
  
  cliente:
    container_name: asir_cliente_web
    image: ubuntu
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 33.33.5.1
    networks:
      red_33:
        ipv4_address: 33.33.5.33

networks:
  red_33:
    external: true


# 5. DIG

Por último arrancaremos los contenedores:

$ docker compose up -d

Entraremos al cliente:

$ docker exec -it cliente_web bash

Haremos un "apt update" e instalaremos el dig con "apt install dnsutils".

Lanzamos el siguiente comando sin "@" y con la IP del servidor DNS:

dig 33.33.5.1 www.fabulasoscuras.int
dig 33.33.5.1 www.fabulasmaravillosas.int

Nos debe devolver la ip a donde apunta el dominio que le pedimos.

# 6. DirectoryIndex y Virtual HOST

Si nos vamos al archivo htpd.confg y buscamos el puerto 80 que se llama "Listen 80", debajo tendremos que poner la configuración para los virtual hosts y sería tal que así:

<VirtualHost *:80>
    DocumentRoot /usr/local/apache2/htdocs/www1
    ServerName www.fabulasmaravillosas.int
</VirtualHost>

La ruta tiene que ser las carpetas con los documentos html y asignarselas a los servers que nosotros queramos, por ejemplo "fabulasmaravillosas" la tengo con www1.

Una vez hecho esto iremos al cliente, descargaremos Lynx con $ apt install lynx y pondremos lo siguiente --> lynx www.fabulasmaravillosas.int:80 

Nos debe resolver el noombre y mostrar lo que tenenmos dentro de nuestra carpeta, ya sea un html o una imagen e.t.c.
