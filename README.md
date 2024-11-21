# Página web con Wordpress - Base de datos y sitio en Docker 
Este proyecto demuestra cómo configurar un sitio de WordPress utilizando Docker, con una configuración personalizada basada en Ubuntu para el contenedor de WordPress. En lugar de usar una imagen preconstruida de WordPress, instalamos y configuramos Apache manualmente dentro del contenedor de Ubuntu. Además, el proyecto incluye un contenedor independiente con MySQL 5.7, que sirve como base de datos para WordPress.

* **Configuración personalizada basada en Ubuntu**: El contenedor de WordPress se construye desde una imagen base de `ubuntu:latest`, permitiendo control total sobre el software instalado.
* **Integración con base de datos MySQL**: Un contenedor independiente ejecuta la base de datos MySQL para WordPress.
* **Configuración personalizada de Apache**: Apache se instala y configura manualmente en el contenedor de WordPress.
* **Facilidad de uso**: Todo se orquesta usando `docker-compose`, lo que permite ejecutar todo con un solo comando.

## docker-compose.yml
El archivo docker-compose.yml define los servicios (WordPress y MySQL) y sus configuraciones.

#### Estructura general

* **`version: '3.8'`**: Especifica la versión de Docker Compose utilizada.
* **`services`**: Define los contenedores de la aplicación.

#### Servicio de WordPress

* **`image: ubuntu:latest`**: Parte de la imagen base más reciente de Ubuntu.
* **`ports`**: Expone el puerto 80 del contenedor en el puerto 8080 del host.
* **`networks`**: Conecta el contenedor a una red personalizada para la comunicación con otros contenedores.
* **`command`**: Ejecuta un comando personalizado al iniciar el contenedor:
  * `apt update`: Actualiza la lista de paquetes.
  * `apt install -y apache2`: Instala Apache.
  * `echo 'ServerName localhost' >> /etc/apache2/apache2.conf`: Agrega un `ServerName` básico para evitar advertencias.
  * `apache2ctl -D FOREGROUND`: Inicia Apache en primer plano para mantener el contenedor ejecutándose.

#### Servicio de MySQL

* **`image: mysql:5.7`**: Usa la imagen oficial de MySQL 5.7.
* **`environment`**: Configura variables de entorno para inicializar la base de datos.
* **`volumes`**: Mapea un volumen para persistir los datos de la base de datos.
* **`networks`**: Conecta la base de datos a la misma red que WordPress.

