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

