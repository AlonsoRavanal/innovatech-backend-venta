# 🛒 Innovatech Backend Ventas

Microservicio REST desarrollado con **Spring Boot 3.4.4** para la gestión de ventas de la empresa Innovatech Chile. Forma parte de un sistema de microservicios junto a `innovatech-backend-despachos` y `innovatech-frontend-despacho`.

---

## 📋 Tabla de contenidos

- [Tecnologías](#tecnologías)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Requisitos previos](#requisitos-previos)
- [Variables de entorno](#variables-de-entorno)
- [Ejecución con Docker](#ejecución-con-docker)
- [Ejecución local](#ejecución-local)
- [Endpoints disponibles](#endpoints-disponibles)
- [Modelo de datos](#modelo-de-datos)
- [Documentación Swagger](#documentación-swagger)

---

## 🛠 Tecnologías

| Tecnología | Versión |
|---|---|
| Java | 17 |
| Spring Boot | 3.4.4 |
| Spring Data JPA | 3.4.3 |
| MySQL Connector | 8.x |
| Lombok | 1.18.36 |
| SpringDoc OpenAPI (Swagger) | 2.7.0 |
| Maven | 3.9.6 |
| Docker | 20.x+ |

---

## 📁 Estructura del proyecto

```
innovatech-backend-ventas/
├── src/
│   └── main/
│       ├── java/com/citt/
│       │   ├── config/             # Configuración de la aplicación (CORS, beans)
│       │   ├── controller/         # Controladores REST
│       │   │   └── VentaController.java
│       │   ├── exceptions/         # Manejo de excepciones globales
│       │   │   ├── VentaNotFoundException.java
│       │   │   └── dto/            # DTOs para respuestas de error
│       │   └── persistence/
│       │       ├── entity/         # Entidades JPA
│       │       │   └── Venta.java
│       │       ├── repository/     # Repositorios Spring Data
│       │       └── services/       # Lógica de negocio
│       │           └── VentaService.java
│       └── resources/
│           └── application.properties
├── Dockerfile
├── docker-compose.yml
└── pom.xml
```

---

## ✅ Requisitos previos

Para ejecutar con Docker:
- [Docker](https://www.docker.com/) 20.x+
- [Docker Compose](https://docs.docker.com/compose/) v2+

Para ejecutar localmente sin Docker:
- Java 17
- Maven 3.9+
- MySQL 8.0 corriendo en `localhost:3306`

---

## 🔐 Variables de entorno

El servicio se configura mediante las siguientes variables de entorno definidas en el `docker-compose.yml`:

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Host de la base de datos | `db` (nombre del servicio Docker) |
| `DB_PORT` | Puerto de MySQL | `3306` |
| `DB_NAME` | Nombre de la base de datos | `db_ventas` |
| `DB_USERNAME` | Usuario de la base de datos | `root` |
| `DB_PASSWORD` | Contraseña de la base de datos | `root_pass` |

Estas variables son leídas por `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://${DB_ENDPOINT}:${DB_PORT}/${DB_NAME}?useSSL=false&serverTimezone=UTC&createDatabaseIfNotExist=true
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

---

## 🐳 Ejecución con Docker

Este es el método recomendado. Levanta el backend junto a su base de datos MySQL con un solo comando.

### 1. Clonar el repositorio

```bash
git clone https://github.com/TU-USUARIO/innovatech-backend-ventas.git
cd innovatech-backend-ventas
```

### 2. Levantar el stack

```bash
docker compose up --build -d
```

### 3. Verificar que los contenedores estén corriendo

```bash
docker compose ps
```

Deberías ver dos contenedores activos:

```
NAME                      STATUS
test-backend-ventas       Up
test-db-ventas            Up (healthy)
```

### 4. Ver logs

```bash
docker compose logs -f backend-ventas
```

### 5. Detener los servicios

```bash
# Detener sin borrar datos
docker compose down

# Detener y eliminar volúmenes (borra la BD)
docker compose down -v
```

---

## 💻 Ejecución local

Si prefieres ejecutar sin Docker, necesitas tener MySQL corriendo localmente.

### 1. Configurar variables de entorno

Crea un archivo `.env` en la raíz o exporta las variables en tu terminal:

```bash
export DB_ENDPOINT=localhost
export DB_PORT=3306
export DB_NAME=db_ventas
export DB_USERNAME=root
export DB_PASSWORD=tu_password
```

### 2. Compilar y ejecutar

```bash
mvn clean package -DskipTests
java -jar target/*.jar
```

O directamente con Maven:

```bash
mvn spring-boot:run
```

---

## 📡 Endpoints disponibles

Base URL: `http://localhost:8080/api/v1/ventas`

| Método | Endpoint | Descripción | Código respuesta |
|---|---|---|---|
| `GET` | `/` | Obtener todas las ventas | 200 |
| `GET` | `/{idVenta}` | Obtener una venta por ID | 200 / 404 |
| `POST` | `/` | Crear una nueva venta | 201 |
| `PUT` | `/{idVenta}` | Actualizar una venta existente | 200 / 404 |
| `DELETE` | `/{idVenta}` | Eliminar una venta por ID | 204 / 404 |

### Ejemplo: Crear una venta

**Request:**
```http
POST http://localhost:8080/api/v1/ventas
Content-Type: application/json
```

```json
{
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "fechaCompra": "2026-06-10",
  "despachoGenerado": false
}
```

**Response `201 Created`:**
```json
{
  "idVenta": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "fechaCompra": "2026-06-10",
  "despachoGenerado": false
}
```

### Ejemplo: Actualizar una venta

**Request:**
```http
PUT http://localhost:8080/api/v1/ventas/1
Content-Type: application/json
```

```json
{
  "despachoGenerado": true
}
```

**Response `200 OK`:**
```json
{
  "idVenta": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 15000,
  "fechaCompra": "2026-06-10",
  "despachoGenerado": true
}
```

---

## 🗃 Modelo de datos

### Entidad `Venta`

| Campo | Tipo | Descripción |
|---|---|---|
| `idVenta` | `Long` | ID autogenerado (PK) |
| `direccionCompra` | `String` | Dirección de entrega (obligatoria) |
| `valorCompra` | `int` | Valor total de la compra |
| `fechaCompra` | `LocalDate` | Fecha de la compra (formato `YYYY-MM-DD`) |
| `despachoGenerado` | `Boolean` | Indica si se generó un despacho (`false` por defecto) |

---

## 📖 Documentación Swagger

Una vez levantado el servicio, la documentación interactiva está disponible en:

```
http://localhost:8080/swagger-ui.html
```

Desde ahí puedes explorar y probar todos los endpoints directamente desde el navegador.

---

## 🏗 Pipeline CI/CD

Este repositorio incluye un pipeline de GitHub Actions que se activa automáticamente al hacer `push` sobre la rama `deploy`.

El pipeline ejecuta los siguientes pasos:
1. **Build** — construye la imagen Docker del servicio
2. **Push** — publica la imagen en el registro de contenedores (ECR o Docker Hub)
3. **Deploy** — despliega la imagen actualizada en la instancia EC2 correspondiente

Las credenciales del registro de imágenes se gestionan como **GitHub Secrets** y nunca se exponen en el código.