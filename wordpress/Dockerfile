# Usar Ubuntu como base
FROM ubuntu:latest

# Actualizar el sistema e instalar las dependencias
RUN apt update && apt upgrade -y
RUN apt install -y apache2 php php-mysql libapache2-mod-php wget unzip curl

# Descargar WordPress e instalar
RUN cd /tmp && \
    wget https://wordpress.org/latest.zip && \
    unzip latest.zip && \
    mv wordpress/* /var/www/html/

# Configurar permisos
RUN chown -R www-data:www-data /var/www/html
RUN chmod -R 755 /var/www/html

# Configuración de Apache para permitir reescritura de URL
RUN echo '<VirtualHost *:80>\n\
    ServerAdmin admin@example.com\n\
    DocumentRoot /var/www/html\n\
    DirectoryIndex index.php\n\
    <Directory /var/www/html>\n\
        AllowOverride All\n\
    </Directory>\n\
    ErrorLog ${APACHE_LOG_DIR}/error.log\n\
    CustomLog ${APACHE_LOG_DIR}/access.log combined\n\
</VirtualHost>' > /etc/apache2/sites-available/000-default.conf

# Habilitar mod_rewrite y reiniciar Apache
RUN a2enmod rewrite
RUN service apache2 restart

# Copiar el script para configurar wp-config.php
COPY setup.sh /setup.sh
RUN chmod +x /setup.sh

# Exponer el puerto 80
EXPOSE 80

# Ejecutar Apache en primer plano
CMD ["/bin/bash", "-c", "apache2ctl -D FOREGROUND"]
