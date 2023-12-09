FROM php:8.1.0-apache

# Para evitar interacciones durante la instalación de paquetes, si no se pone cuando la terminal pregunte una confirmación se interrumpe la creación de la imagen.
ARG DEBIAN_FRONTEND=noninteractive

# Paquetes necesarios de la imagen.
RUN apt-get update && apt-get install -y \
        libpng-dev \
        libzip-dev \
        zlib1g-dev \
        libonig-dev \
        vim \
        sendmail \
    && docker-php-ext-install mysqli mbstring zip gd \
    && a2enmod rewrite \
    && a2enmod proxy \
    && a2enmod proxy_http \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


# Copiar archivos de configuración de virtual host
COPY ./conf/*.conf /etc/apache2/sites-available/

# Copiar archivos de contraseñas
COPY ./etc/apache2/.htpasswd /etc/apache2/.htpasswd

# Copiar archivos de contraseñas
COPY ./certs /etc/apache2/ssl

# Habilitar el módulo SSL
RUN a2enmod ssl

# Habilitar los virtual hosts
RUN a2ensite intranet.conf ricardo-fernandez-guzman-phpmyadmin.conf



