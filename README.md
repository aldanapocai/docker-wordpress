# Página web con Wordpress - Base de datos y sitio en Docker 
Este proyecto demuestra cómo configurar un sitio de WordPress utilizando Docker, con una configuración personalizada basada en Ubuntu para el contenedor de WordPress. En lugar de usar una imagen preconstruida de WordPress, instalamos y configuramos Apache manualmente dentro del contenedor de Ubuntu. Además, el proyecto incluye un contenedor independiente con MySQL 5.7, que sirve como base de datos para WordPress.

## Estructura del Proyecto
El proyecto incluye los siguientes archivos:

### 1. `Dockerfile`
Define cómo construir el contenedor de WordPress desde cero, utilizando Ubuntu como base e instalando Apache, PHP y las extensiones necesarias.
### 2. `docker-compose.yml`
Orquesta los servicios necesarios para ejecutar WordPress, incluyendo:
- Un servicio para WordPress construido a partir del `Dockerfile`.
- Un servicio para la base de datos MySQL, con configuración automatizada.


---


## Explicación del `Dockerfile`
El `Dockerfile` es responsable de construir el contenedor que ejecutará WordPress.
A continuación se explican las instalaciones necesarias, que se hacen de forma automatizada en el Dockerfile.

Para eso, vamos al sitio web <https://wordpress.org/about/requirements/>, que son los requerimientos de wordpress.
Allí vemos que necesita el lenguaje php, la base de datos MySQL (o MariaDB), y un web server (Apache o Nginx) que soporte PHP y MySQL. 
Para más información de cada uno, visitar <https://make.wordpress.org/hosting/handbook/server-environment/>

### `Apache`

Apache es un servidor web disponible para Linux, sin cargo.

La instalación es sencilla, es simplemente:
```bash
  sudo apt install apache2
```

Más información sobre esta instalación en este tutorial de ubuntu tutorials: <https://ubuntu.com/tutorials/install-and-configure-apache#3-creating-your-own-website>

Por defecto, Apache buscará en la ruta /var/www/html/ los archivos web a mostrar en el sitio web.

### `PHP`

PHP es un lenguaje de scripting de propósito general muy usado para desarrollo web. PHP siempre va a estar asociado a un servidor web, por lo que también se deben instalar módulos para la comunicación entre php y el servidor web. Simplemente correr:

```bash
  sudo apt install php 

  sudo apt install libapache2-mod-php
```
Además, si se planea usar como base de datos a MySQL, se necesita instalar el paquete PHP-MYSQL:

```bash
  sudo apt install php-mysql
```

Para más información, consultar el tutorial de Ubuntu sobre PHP y web servers: <https://ubuntu.com/server/docs/how-to-install-and-configure-php> 

Con esto ya cubrimos los requerimientos de extensiones php que solicita wordpress en su documentación de server enviroment (<https://make.wordpress.org/hosting/handbook/server-environment/>), que son básicamente:

 - json (pero se aclara que está “bundled” (incluida) en php mayor a 8.0.0, por lo que no es necesario).
 - mysqli o mysqlnd, que sirven para conectar PHP con MySQL. Por esto instalamos el módulo **php-mysql.**

Finalmente, necesitaremos *wget* y *unzip* para pasos posteriores. Entonces, todas las instalaciones necesarias se pueden completar con la siguiente línea en nuestro *Dockerfile*:
```bash
  RUN apt install -y apache2 php php-mysql libapache2-mod-php wget unzip
```

### **Instalación de WordPress en sí**

Tomamos como referencia a este tutorial oficial de WordPress: **(<https://developer.wordpress.org/advanced-administration/before-install/howto-install> )**
Estos son los pasos que dice:
1) Descargar y descomprimir el Wordpress package. Por esto necesitamos instalar **wget** y **unzip**.
2) Crear la base de datos. Lo hacemos con imagen de MySQL directa.
3) Configurarla, con wp-config.php. Igual, si no lo hacemos, se configura de forma gráfica.
4) Volcar los archivos descomprimidos de WordPress en el directorio raiz del web server (/var/www/html para Apache)
5) Entrar a la página web en sí para ejecutar el script de instalación de Wordpress

Para cumplir con todas estos pasos, utilizamos la siguiente línea en nuestro *Dockerfile*:
```bash
  RUN cd /home &&  wget https://wordpress.org/latest.zip && unzip latest.zip && mv wordpress/\* /var/www/html/

  RUN rm -rf /home/latest.zip /home/wordpress
```
Este comando comienza moviéndose a home. Luego descarga, descomprime, y mueve la carpeta generada *wordpress* a */var/www/html* que es donde se va a ejecutar todo.
Luego, borra la descarga y la carpeta resultante (que quedó vacía).
No extraer directamente es /var/www/html, ya que es una mala práctica.

- **Configuración de Virtual Hosts**

Los “virtual hosts” se utilizan para hacer host de más de un solo dominio web en el mismo server. Allí se pueden encapsular configuraciones para cada dominio.

Apache, por default, viene configurado para tener un solo dominio, que sirve a los archivos ubicados en /var/www/html

Apache necesita tener el ownership del directorio donde va a operar, y los permisos suficientes. Para ello nuestro *Dockerfile* ejecuta:

```bash
  RUN chown -R www-data:www-data /var/www/html

  RUN chmod -R 755 /var/www/html
```

Así apache puede modificar estos archivos, y los grupos y otros únicamente podrán leer y ejecutar. Más información en este link: <https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04> , apartado “Setting up virtual hosts”

Finalmente, tenemos que configurar el archivo de configuración de Virtual Hosts,  000-default.conf

Para ello, en nuestro *Dockerfile*, vamos a poner esto:

```bash
RUN echo '<VirtualHost \*:80>\n\

    ServerAdmin martin0peni@gmail.com\n\

    DocumentRoot /var/www/html\n\

    DirectoryIndex index.php\n\

    <Directory /var/www/html>\n\

        AllowOverride All\n\

    </Directory>\n\

    ErrorLog ${APACHE\_LOG\_DIR}/error.log\n\

    CustomLog ${APACHE\_LOG\_DIR}/access.log combined\n\

</VirtualHost>' > /etc/apache2/sites-available/000-default.conf
```
Esto hace lo siguiente:
1) El virtual host sirve al puerto 80.
2) Document root: Es dónde se buscan el archivo html de la página web. Será /var/www/html.
3) Directory index: Es el archivo por defecto que se busca para ejecutar como página web. Será index.php 
4) Se hace un Allow Override (probar quitarlo)
5) Error log y custom log: son los archivos de logging. Dejamos ubicación por defecto.

### **Configuraciones adicionales**

- **`RUN a2enmod rewrite'`**:
  - Este comando habilita el módulo `mod_rewrite` en Apache, el cual es necesario para que WordPress pueda gestionar permalinks. Más información en la [documentación oficial de mod_rewrite](https://httpd.apache.org/docs/current/mod/mod_rewrite.html).
 
- **`RUN service apache2 restart`**:
  - Reinicia el servicio de Apache para aplicar los cambios realizados, como la habilitación de módulos o la modificación de configuraciones.
    
- **`COPY setup.sh /setup.sh`**:
  - Copia el archivo `setup.sh` desde el sistema host al contenedor en la ruta `/setup.sh`. Este script es útil para automatizar tareas como la configuración del archivo `wp-config.php`, que define la conexión a la base de datos y otros parámetros clave para WordPress.
- **`RUN chmod +x /setup.sh`**:
  - Otorga permisos de ejecución al script copiado para que pueda ejecutarse dentro del contenedor.
    
- **`EXPOSE 80`**:
  - Declara que el contenedor utiliza el puerto 80 para manejar solicitudes HTTP. Este comando no configura el puerto, pero es informativo para herramientas como Docker Compose. Más detalles en la [documentación de EXPOSE](https://docs.docker.com/engine/reference/builder/#expose).
 
- **`CMD ["/bin/bash", "-c", "apache2ctl -D FOREGROUND"]`**:
  - Inicia Apache en primer plano (`-D FOREGROUND`) para que el proceso principal del contenedor siga activo. Esto es esencial para evitar que Docker detenga el contenedor cuando no hay procesos en ejecución. Más información en la [documentación de CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## Explicación del `docker-compose.yml`

- **`version: '3.8'`**:
  - Define la versión de Docker Compose utilizada, que en este caso es la 3.8. Esta versión es compatible con funcionalidades modernas de Docker Compose.

#### Servicio `wordpress`:
- **`build:`**:
  - **`context: .`**:
    - Indica que el contenedor de WordPress debe construirse utilizando el archivo `Dockerfile` ubicado en el directorio actual.
- **`ports:`**:
  - **`"8080:80"`**:
    - Mapea el puerto 80 del contenedor (Apache) al puerto 8080 del host, permitiendo acceder a WordPress desde el navegador en `http://localhost:8080`.
- **`depends_on:`**:
  - **`db`**:
    - Define que el contenedor de WordPress depende del servicio `db` (base de datos). Esto asegura que el contenedor de MySQL se inicie antes de que WordPress intente conectarse.
- **`networks:`**:
  - **`wordpress_network`**:
    - Conecta el contenedor de WordPress a una red personalizada para que pueda comunicarse con otros servicios (como la base de datos).

#### Servicio `db` (Base de Datos):
- **`image: mysql:5.7`**:
  - Especifica que se utilizará la imagen oficial de MySQL versión 5.7.
- **`environment:`**:
  - Configura las credenciales de la base de datos mediante variables de entorno:
    - **`MYSQL_ROOT_PASSWORD`**:
      - Contraseña para el usuario root de la base de datos.
    - **`MYSQL_DATABASE`**:
      - Nombre de la base de datos que se creará automáticamente (en este caso, `wordpress`).
    - **`MYSQL_USER`**:
      - Usuario de la base de datos que WordPress utilizará para conectarse.
    - **`MYSQL_PASSWORD`**:
      - Contraseña para el usuario configurado.
- **`volumes:`**:
  - **`db_data:/var/lib/mysql`**:
    - Define un volumen para almacenar los datos de la base de datos. Esto asegura que los datos persistan incluso si el contenedor se reinicia o elimina.
- **`networks:`**:
  - **`wordpress_network`**:
    - Conecta la base de datos a la misma red que el contenedor de WordPress.

