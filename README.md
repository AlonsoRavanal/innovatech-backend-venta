# 🛒 Innovatech Backend Ventas

Microservicio REST desarrollado con **Spring Boot 3.4.4** para la gestión de ventas de la empresa Innovatech Chile. Forma parte de un sistema de microservicios junto a `innovatech-backend-despachos` y `innovatech-frontend-despachos`.

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
- [Pipeline CI/CD](#-pipeline-cicd)
- [Despliegue en Kubernetes](#-despliegue-en-kubernetes)
- [Verificación y monitoreo del despliegue](#-verificación-y-monitoreo-del-despliegue)

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
| Kubernetes (K3s) | 1.x |

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

Además, en el servidor de despliegue se mantiene un repositorio paralelo (`/repository/user7/EP3`) donde conviven los tres proyectos clonados junto con los manifiestos de Kubernetes:

```
/repository/user7/EP3/
├── innovatech-backend-venta/
├── innovatech-backend-despacho/
├── innovatech-frontend-despachos/
├── deployment.yaml
├── frontend-deployment.yaml
├── db-deployment.yaml
├── service.yaml
├── frontend-service.yaml
├── pvc.yaml
├── secrets.yaml
├── hpa.yaml
├── pipeline-local.sh
└── README.md
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

Para desplegar en Kubernetes:
- Clúster K3s local operativo (verificado mediante `kubectl config current-context`)
- `kubectl` configurado con permisos suficientes (`kubectl auth can-i create deployments`)
- Registro de imágenes local disponible en `localhost:5000`
- MetalLB (o un proveedor de LoadBalancer equivalente) para exponer el frontend

---

## 🔐 Variables de entorno

El servicio se configura mediante las siguientes variables de entorno definidas en el `docker-compose.yml` (entorno local) o en los manifiestos de Kubernetes (entorno de clúster):

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Host de la base de datos | `db` (Docker) / `db-ventas` (Kubernetes) |
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

En Kubernetes, `DB_USERNAME` y `DB_PASSWORD` no se definen como texto plano: se inyectan desde el recurso `Secret` `mysql-secrets` mediante `secretKeyRef`, evitando exponer credenciales en los manifiestos.

---

## 🐳 Ejecución con Docker

Este es el método recomendado para desarrollo local. Levanta el backend junto a su base de datos MySQL con un solo comando.

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

---

## ☸️ Despliegue en Kubernetes

Para desplegar la solución completa (dos backends, frontend y bases de datos) en un clúster K3s se elaboró un conjunto de manifiestos YAML declarativos, ubicados fuera de los directorios de cada proyecto, en la raíz del repositorio de trabajo (`/repository/user7/EP3`).

### Recursos definidos

| Archivo | Recurso(s) | Propósito |
|---|---|---|
| `deployment.yaml` | `Deployment` backend-venta / backend-despacho | Define imagen, réplicas, variables de entorno, límites de recursos y `readinessProbe` de ambos backends |
| `frontend-deployment.yaml` | `Deployment` frontend-despachos | Define el contenedor del frontend (imagen, puerto 8080, límites de recursos) |
| `db-deployment.yaml` | `Deployment` + `Service` mysql-ventas / mysql-despachos | Instancias MySQL independientes por dominio, con almacenamiento persistente |
| `service.yaml` | `Service` (ClusterIP) backend-ventas / backend-despachos | Direcciones internas estables para la comunicación entre componentes |
| `frontend-service.yaml` | `Service` (LoadBalancer) frontend-lb-svc | Expone el frontend hacia el exterior del clúster en el puerto 80 |
| `pvc.yaml` | `PersistentVolumeClaim` mysql-ventas-pvc / mysql-despachos-pvc | Almacenamiento persistente (256Mi, `local-path`) para cada base de datos |
| `secrets.yaml` | `Secret` mysql-secrets | Credenciales de base de datos (`db-user`, `db-pass`), inyectadas vía `secretKeyRef` |
| `hpa.yaml` | `HorizontalPodAutoscaler` backend-venta-hpa / backend-despacho-hpa | Autoescalado de 1 a 3 réplicas por backend, con objetivo de 60% de uso de CPU |

### Detalles de configuración

- Cada backend expone su puerto de contenedor (`8080` para ventas, `8081` para despachos) enrutado internamente por su respectivo `Service` ClusterIP.
- El frontend resuelve los backends mediante los nombres DNS internos que gestiona CoreDNS (`http://backend-ventas:8080`, `http://backend-despachos:8081`), sin depender de IPs dinámicas de pods.
- El `Service` `frontend-lb-svc` (tipo `LoadBalancer`) escucha en el puerto estándar HTTP (80) y redirige al `targetPort: 8080` del contenedor del frontend.
- Cada `Deployment` de backend define `resources.requests`/`limits` de CPU y memoria, además de un `readinessProbe` tipo `tcpSocket` antes de recibir tráfico.
- Las credenciales de MySQL no se escriben en texto plano en los manifiestos de despliegue: se referencian desde el `Secret` `mysql-secrets`.

### Automatización del despliegue: `pipeline-local.sh`

Se desarrolló el script `pipeline-local.sh` para automatizar el ciclo completo de despliegue local:

1. Construye las tres imágenes Docker (`backend-venta`, `backend-despacho`, `frontend-despachos`).
2. Publica las imágenes en el registro local (`localhost:5000`).
3. Aplica todos los manifiestos de Kubernetes mediante `kubectl apply` (`secrets.yaml`, `pvc.yaml`, `db-deployment.yaml`, `service.yaml`, `frontend-service.yaml`, `deployment.yaml`, `frontend-deployment.yaml`, `hpa.yaml`).
4. Espera a que cada `Deployment` complete su rollout (`kubectl rollout status`).
5. Muestra el estado final de Pods, Servicios, HPA y PVC.

```bash
chmod +x pipeline-local.sh
./pipeline-local.sh
```

---

## 🔎 Verificación y monitoreo del despliegue

Comandos utilizados para validar que la infraestructura quedó completamente operativa:

```bash
# Contexto y permisos del clúster
kubectl config current-context
kubectl auth can-i create deployments

# Estado general de los recursos
kubectl get deploy,svc,hpa,pods -o wide

# Prueba de acceso al frontend (IP externa provista por MetalLB)
curl http://<IP-EXTERNA-FRONTEND>

# Comunicación interna frontend → backends
kubectl logs -f pod/<frontend-pod>

# Logs de cada componente
kubectl logs deploy/frontend-despachos --tail=30
kubectl logs deploy/backend-venta --tail=30
kubectl logs deploy/backend-despacho --tail=30

# Historial de eventos del namespace
kubectl get events --sort-by=.lastTimestamp

# Métricas de consumo y autoescalado
kubectl top pods
kubectl get hpa
kubectl describe hpa backend-venta
kubectl describe hpa backend-despacho

# Prueba de recuperación ante reinicio controlado
kubectl rollout restart deployment/backend-venta
kubectl rollout status deployment/backend-venta
kubectl get pods
```

Para visualizar el frontend desde un equipo externo al servidor, se estableció un túnel SSH con reenvío de puertos:

```bash
ssh -L 8080:<IP-EXTERNA-FRONTEND>:80 usuario@servidor
```

Esto permite acceder a la aplicación en `http://localhost:8080` sin exponer directamente la red interna del servidor.

Las pruebas funcionales realizadas confirmaron:
- Creación de una venta vía Postman (`POST /api/v1/ventas` → `201 Created`).
- Visualización de la venta creada desde el frontend.
- Generación de un despacho asociado a la venta desde el frontend.
- Confirmación del despacho creado consultando el backend de despachos vía Postman (`GET /api/v1/despachos` → `200 OK`).