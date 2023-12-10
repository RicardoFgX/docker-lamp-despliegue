# Configuración de la Pila LAMP en Docker

# Configuración de la Pila LAMP en Docker

## Índice

1. [Requisitos Previos](#requisitos-previos)
2. [PARTE 1: Configuración de Virtual Host](#parte-1-configuración-de-virtual-host)
    1. [Modificación de Nombres de VirtualHost](#1-modificación-de-nombres-de-virtualhost)
        - [1.1. Acceder al Archivo de Hosts](#11-acceder-al-archivo-de-hosts)
        - [1.2. Editar el Archivo de Hosts](#12-editar-el-archivo-de-hosts)
        - [1.3. Acceder al Archivo de Configuración de Apache](#13-acceder-al-archivo-de-configuración-de-apache)
        - [1.4. Editar el Archivo de Configuración](#14-editar-el-archivo-de-configuración)
        - [1.5. Reinicia Docker-Compose](#15-reinicia-docker-compose)
    2. [Creación de Virtual Host para PhpMyAdmin](#2-creación-de-virtual-host-para-phpmyadmin)
    3. [Modificación del Index.html de Intranet](#3-modificación-del-indexhtml-de-intranet)
    4. [Añadir Nuevo Usuario](#4-añadir-nuevo-usuario)
    5. [Instalación de CMS Wordpress](#5-instalación-de-cms-wordpress)
3. [PARTE 2: Instalación de Certificados SSL](#parte-2-instalación-de-certificados-ssl)
    1. [Generación de Certificados](#1-generación-de-certificados)
        - [2.1.1. Crear Directorio de Certificados](#11-crear-directorio-de-certificados)
        - [2.1.2. Lanzar el Comando de Generación de Certificados](#12-lanzar-el-comando-de-generación-de-certificados)
        - [2.1.3. Especificar Common Name](#13-especificar-common-name)
    2. [Configurar Virtual Host 443](#2-configurar-virtual-host-443)
        - [2.2.1. Editar el Archivo de Configuración](#22-editar-el-archivo-de-configuración)
    3. [Habilitar Módulo mod_ssl](#3-habilitar-módulo-mod_ssl)

## Requisitos Previos

Antes de comenzar con este procedimiento, partimos de haber completado correctamente los pasos para el despliegue del servidor web Apache cedido por el profesor. Puedes encontrar las instrucciones detalladas para hacerlo en el archivo readme.md del repositorio [docker-lamp](https://github.com/antonio-gabriel-gonzalez-casado/docker-lamp/).

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

Encuentra y abre el archivo de configuración de Apache del docker. Para nuestra estructura de archivos seria en:

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

#### 1.5. Reinicia Docker-Compose
Se reinicia docker-compose para aplicar los cambios:
```
docker-compose restart
```
A continuación ya se deberia de poder acceder utilizando las siguientes URLs

http://ricardo-fernandez-guzman-www.local

![www.local](docs/images/imagen4.png)

http://ricardo-fernandez-guzman-intranet.local

![intranet.local](docs/images/imagen5.png)

### 2. Creación de Virtual Host para PhpMyAdmin

### 3. Modificación del Index.html de Intranet

### 4. Añadir Nuevo Usuario

### 5. Instalación de CMS Wordpress

## PARTE 2: Instalación de Certificados SSL

### 1. Generación de Certificados

#### 1.1 Crear Directorio de Certificados
#### 1.2 Lanzar el Comando de Generación de Certificados
#### 1.3 Editar el Archivo de Configuración

### 2. Configurar Virtual Host 443

### 3. Habilitar Módulo mod_ssl