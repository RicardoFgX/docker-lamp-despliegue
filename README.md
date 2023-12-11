# Configuración de la Pila LAMP en Docker

## Índice

1. [Requisitos Previos](#requisitos-previos)
2. [Estructura del Proyecto](#estructura-del-proyecto)
3. [PARTE 1: Configuración de Virtual Host](#parte-1-configuración-de-virtual-host)
    1. [Modificación de Nombres de VirtualHost](#1-modificación-de-nombres-de-virtualhost)
        - [1.1. Acceder al Archivo de Hosts](#11-acceder-al-archivo-de-hosts)
        - [1.2. Editar el Archivo de Hosts](#12-editar-el-archivo-de-hosts)
        - [1.3. Acceder al Archivo de Configuración de Apache](#13-acceder-al-archivo-de-configuración-de-apache)
        - [1.4. Editar el Archivo de Configuración](#14-editar-el-archivo-de-configuración)
        - [1.5. Reiniciar Docker-Compose y comprobación](#15-reiniciar-docker-compose-y-comprobación)
    2. [Creación de Virtual Host para PhpMyAdmin](#2-creación-de-virtual-host-para-phpmyadmin)
    3. [Modificación del Index.html de Intranet](#3-modificación-del-indexhtml-de-intranet)
    4. [Añadir Nuevo Usuario](#4-añadir-nuevo-usuario)
    5. [Instalación de CMS Wordpress](#5-instalación-de-cms-wordpress)
4. [PARTE 2: Instalación de Certificados SSL](#parte-2-instalación-de-certificados-ssl)
    1. [Generación de Certificados](#1-generación-de-certificados)
        - [2.1.1. Crear Directorio de Certificados](#11-crear-directorio-de-certificados)
        - [2.1.2. Lanzar el Comando de Generación de Certificados](#12-lanzar-el-comando-de-generación-de-certificados)
        - [2.1.3. Especificar Common Name](#13-especificar-common-name)
    2. [Configurar Virtual Host 443](#2-configurar-virtual-host-443)
        - [2.2.1. Editar el Archivo de Configuración](#22-editar-el-archivo-de-configuración)
    3. [Habilitar Módulo mod_ssl](#3-habilitar-módulo-mod_ssl)

## Requisitos Previos

Antes de comenzar con este procedimiento, partimos de haber completado correctamente los pasos para el despliegue del servidor web Apache cedido por el profesor. Puedes encontrar las instrucciones detalladas para hacerlo en el archivo readme.md del repositorio [docker-lamp](https://github.com/antonio-gabriel-gonzalez-casado/docker-lamp/).

## Estructura del proyecto
La estructura del proyecto finalizado es la siguiente:
```
docker-lamp-despliegue
├─ .gitignore 
├─ LICENSE
├─ README.md
├─ apache2-php
│  ├─ certs
│  │  ├─ intranet.crt
│  │  ├─ intranet.key
│  │  ├─ phpmyadmin.crt
│  │  ├─ phpmyadmin.key
│  │  ├─ www.crt
│  │  └─ www.key
│  ├─ Dockerfile
│  ├─ conf
│  │  ├─ 000-default.conf
│  │  ├─ intranet.conf
│  │  └─ ricardo-fernandez-guzman-phpmyadmin.conf
│  ├─ etc
│  │  └─ apache2
│  │     └─ .htpasswd
│  └─ www
│     ├─ index.html
│     ├─ intranet
│     │  └─ index.html
│     ├─ phpinfo.php
│     ├─ test-bd.php
│     └─ wordpress
│        └─ carpetas del entorno...
├─ dist
│  ├─ env.dist
│  └─ htpasswd.dist
├─ docker-compose.yml
├─ docs
│  └─ images
└─ mysql
   ├─ conf
   └─ dump
      └─ myDb.sql

```

## PARTE 1: Configuración de Virtual Host

### 1. Modificación de Nombres de VirtualHost

Modifica los nombres de los VirtualHost dados de ejemplo:

- www.local -> ricardo-fernandez-guzman-www.local
- intranet.local -> ricardo-fernandez-guzman-intranet.local

#### 1.1. Acceder al Archivo de Hosts

Abrir el archivo de hosts en el sistema. Este archivo generalmente se encuentra en:

- **Windows:** C:\Windows\System32\drivers\etc\hosts
- **Linux/Mac:** /etc/hosts

En mi caso estoy usando windows por lo que se llegaría a esta carpeta:
![arhivoHost](docs/images/imagen1.png)

#### 1.2. Editar el Archivo de Hosts

Se añaden las siguientes líneas al final del archivo, para ello se accede con un editor de texto, por ejemplo el bloc de notas:

```
127.0.0.1    ricardo-fernandez-guzman-www.local
127.0.0.1    ricardo-fernandez-guzman-intranet.local
```
![arhivoHostInside](docs/images/imagen2.png)

Se guarda y se cierra el archivo.

#### 1.3. Acceder al Archivo de Configuración de Apache

Nos dirigimos al archivo de configuración de nuestras páginas del servidor web Apache en nuestro proyecto Docker. Para nuestra estructura de archivos seria en:

- **apache2-php/conf/000-default.conf:** para ricardo-fernandez-guzman-www.local
- **apache2-php/conf/intranet.conf:** para ricardo-fernandez-guzman-intranet.local

![estructura](docs/images/imagen3.png)

#### 1.4. Editar el Archivo de Configuración

Busca las líneas relacionadas con los VirtualHost y modifica los ServerName para reflejar tus nuevos nombres.

##### Para ricardo-fernandez-guzman-www.local

```
<VirtualHost *:80>
    ServerName ricardo-fernandez-guzman-www.local
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
##### Para ricardo-fernandez-guzman-intranet.local
```
Listen 8060

<VirtualHost *:8060>
    ServerName ricardo-fernandez-guzman-intranet.local
    DocumentRoot /var/www/html/intranet

    <Directory /var/www/html/intranet>
        Options Indexes FollowSymLinks
        AllowOverride All

        AuthType Basic
        AuthName "Area Restringida"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

#### 1.5. Reiniciar Docker-Compose y comprobación
Se reinicia docker-compose para aplicar los cambios:
```
docker-compose restart
```
A continuación ya se deberia de poder acceder utilizando las siguientes URLs

http://ricardo-fernandez-guzman-www.local

![www.local](docs/images/imagen4.png)

http://ricardo-fernandez-guzman-intranet.local:8060
Para este caso, debido a la configuración que hemos establecido, es necesario loguearse con un usuario y contraseña válidos. Estos serían los dispuestos en el archivo htpasswd.dist. Los cuales siguen la estructura **usuario:contraseña-cifrada**.

En este caso hemos entrado con el usuario configurado por defecto que seria usuario1 y contraseña 123456789.

![login-intranet.local](docs/images/imagen5.png)

Una vez verificadas las credenciales podremos acceder a la página de esta URL

![login-intranet.local](docs/images/imagen6.png)

### 2. Creación de Virtual Host para PhpMyAdmin
Lo primero que tenemos que hacer es crear un nuevo archivo de configuración del virtual host que se llame ricardo-fernandez-guzman(el ejercicio pide nombre-apellidos)-phpmyadmin.local:8081.

La estructura de nuestro archivo de configuración es muy similar a la de nuestro archivo de configuración de intranet con la diferencia de que en vez de utilizar la etiquea *"directory"* se utiliza *"location"* ya que se va a configurar un proxy inverso para redirigir.
```
Listen 8081

<VirtualHost *:8081>
    ServerName ricardo-fernandez-guzman-phpmyadmin.local

    <Location />
        Options Indexes FollowSymLinks
        AllowOverride All

        AuthType Basic
        AuthName "Area Restringida"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Location>

    ProxyPreserveHost On
    ProxyPass / http://phpmyadmin:80/
    ProxyPassReverse / http://phpmyadmin:80/
</VirtualHost>
```
### 3. Modificación del Index.html de Intranet
Para ello hay que modificar el archivo que se encuentra en [apache2-php/www/intranet/index.html](apache2-php/www/intranet/index.html) y cambiarlo como uno prefiera. En mi caso, ahora la página princiapal para intranet es la siguiente:

![NuevoHtmlIntranet](docs/images/imagen7.png)

### 4. Añadir Nuevo Usuario

Nos piden añadir un nuevo usuario que cuente con la estructura nombre-apellidos a la lista de usuarios que pueden acceder a intranet. Para ello moficaremos el archivo donde se guardan los usuarios y contraseñas([aqui](apache2-php/etc/apache2/.htpasswd)).

Como mencioné antes los usuarios siguen la estructura de usuario-contraseña-cifrada, para realizar esto manera sencilla se ha empleado la página web [hellotools](https://hellotools.org/es/generar-cifrar-contrasena-para-archivo-htpasswd).

No se me ha permitido utilizar guiones en el usuario por lo que se ha optado por la estructura de nombres y apellidos toda junta.

![NuevoHtmlIntranet](docs/images/imagen8.png)

El usuario con la contraseña cifrada sería tal que así:
```
ricardofernandezguzman:$2y$10$PBhaC1jM.yG8Ty7g4fQYaellCkJvg2BeTxhokNGADvv5v8/bJhn4m
```
Para que se apliquen todos estos cambios habría que resetear apache, aunque a mi por algún motivo con eso no me ha funcionado y he tenido que volver a buildear el docker-compose con el comando 
```
docker-compose restart
```
![Docker-compose-Restart](docs/images/imagen12.png)

A continuación se puede entrar en intranet con el nuevo usuario con su respectiva contraseña:

![Docker-compose-Restart](docs/images/imagen10.png)
![Docker-compose-Restart](docs/images/imagen11.png)

### 5. Instalación de CMS Wordpress
Lo primero es acceder al menú de phpMyAdmin utilizando el usuario y contraseña configurados. Al hacerlo entraremos en una ventana similar a la siguiente. El primer paso será ir a la sección para ver todas las bases de datos, se encuentra en el header superior.

![Docker-compose-Restart](docs/images/imagen13.png)

Crearemos una base de datos que será la que utilizaremos para el WordPress, yo no me he comido mucho la cabeza y la he llamado WordPress
![Docker-compose-Restart](docs/images/imagen14.png)

Lo siguiente es crear a un usuario, para ello nos vamos al menú principal de phpMyAdmin en la sección de Cuentas de usuarios

![Docker-compose-Restart](docs/images/imagen15.png)

Aparecen todos los usuarios que ya tenemos, pero como lo que vamos a hacer es uno nuevo le damos al enlace que está debajo de estos usuarios que dice "Agregar cuenta de usuario"

![Docker-compose-Restart](docs/images/imagen16.png)

En este formulario es obligatorio rellenar el nombre, que le hemos puesto wordPress y una contraseña recomendable una segura y activar el tick para otorgarle todos los privilegios

![Docker-compose-Restart](docs/images/imagen17.png)

Volvemos a la ventana con todos los usuarios y ahora vemos que aparece nuestro usuario "wordpress" y vamos a editar los privilegios de este usuario

![Docker-compose-Restart](docs/images/imagen18.png)

Le damos en la parte superior en donde pone "Base de datos" porque vamos a configurar los permisos de este usuario en la base de datos wordpress

![Docker-compose-Restart](docs/images/imagen19.png)
![Docker-compose-Restart](docs/images/imagen20.png)

 A continuación nos aparecerá un menu para asignarle permisos sobre esa base de datos, similar como parece en la configuración de permisos global. Le asignamos todos los permisos dandole al tick que nos permite esta opción.
 Esto sería todo lo que hay que configurar en el phpMyAdmin.

 En nuestro proyecto, lo primero sería descargar WordPress, se puede hacer desde la página Oficial de [WordPress](https://es.wordpress.org/download/)

![Docker-compose-Restart](docs/images/imagen21.png)

Una vez hecho, guardaremos la carpeta de WordPress en ```apache2-php/www``` .Quedaría de la siguiente forma:

![Docker-compose-Restart](docs/images/imagen26.png)

Dentro de esta carpeta hay que modificar el archivo ```wp-config.php``` y asignarles los valores de nuestra base de datos creada en phpMyAdmin en este archivo de configuración; nombre de la base de datos, usuario, contraseña y el host. Este último se puede encontrar en la parte superior de phpMyAdmin.

![Docker-compose-Restart](docs/images/imagen27.png)

Mi configuración es esta:

![Docker-compose-Restart](docs/images/imagen22.png)

En principio ya tenemos todo configurado, para que se actualice todas estas configuraciones realizamos un reset a nuestro docker compose ```docker-compose restart```

Una vez hecho esto, para proceder a la instalación de WordPress, utilizamos una URL con la estructura:
http://example.com/wp-admin/install.php

En mi caso además hay que añadirle la ruta ```/wordpress``` antes de ```/wp-admin``` ya que de normal estamos ubicados en la carpeta ```/www```. Mi URL para acceder a la instalación de WordPress sería:
http://ricardo-fernandez-guzman-intranet.local/wordpress/wp-admin/install.php

Ya solo queda configurar con la información que se quiera el título, así como el usuario y la contraseña, que ya viene generada de base una segura, además del email. Guardamos en un bloc de notas aparte el usuario y la contraseña para no perderlos. Una vez rellenado este formulario le damos al botón de instalar WordPress.

![Docker-compose-Restart](docs/images/imagen23.png)

Al acabar podremos logearnos con el usuario y la contraseña definidos anteriormente

![Docker-compose-Restart](docs/images/imagen24.png)

![Docker-compose-Restart](docs/images/imagen25.png)

## PARTE 2: Instalación de Certificados SSL

### 1. Generación de Certificados

#### 1.1 Crear Directorio de Certificados
#### 1.2 Lanzar el Comando de Generación de Certificados
#### 1.3 Editar el Archivo de Configuración

### 2. Configurar Virtual Host 443

### 3. Habilitar Módulo mod_ssl