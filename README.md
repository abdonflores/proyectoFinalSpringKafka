Proyecto Final: E-commerce con Spring Boot y Kafka
📋 Descripción del Proyecto
Sistema de e-commerce distribuido desarrollado con microservicios utilizando Spring Boot, Apache Kafka para mensajería asíncrona, y PostgreSQL como base de datos. El sistema gestiona productos, órdenes e inventario de manera desacoplada mediante eventos.
🏗️ Arquitectura del Sistema
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  Product        │         │  Order          │         │  Inventory      │
│  Service        │         │  Service        │         │  Service        │
│  (Port 8080)    │         │  (Port 8081)    │         │  (Port 8082)    │
└────────┬────────┘         └────────┬────────┘         └────────┬────────┘
         │                           │                           │
         │    ┌──────────────────────┴───────────────────────┐   │
         │    │            Apache Kafka                      │   │
         └────┤  Topics:                                     ├───┘
              │  - ecommerce.products.created                │
              │  - ecommerce.orders.placed                   │
              │  - ecommerce.orders.confirmed                │
              │  - ecommerce.orders.cancelled                │
              └──────────────────────────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
┌────────▼────────┐ ┌──────▼──────┐ ┌───────▼────────┐
│   PostgreSQL    │ │ PostgreSQL  │ │  PostgreSQL    │
│   Products      │ │   Orders    │ │  Inventory     │
│   (Port 5435)   │ │ (Port 5436) │ │  (Port 5437)   │
└─────────────────┘ └─────────────┘ └────────────────┘
🛠️ Stack Tecnológico

Backend: Java 17+, Spring Boot 3.x
Mensajería: Apache Kafka (Confluent Platform)
Base de Datos: PostgreSQL 15
Containerización: Docker, Docker Compose
Build Tool: Maven
Documentación API: Postman

📦 Microservicios
1. Product Service (Puerto 8080)

Gestión completa de productos (CRUD)
Gestión de categorías
Publicación de eventos ecommerce.products.created
Base de datos: ecommerce_products

2. Order Service (Puerto 8081)

Creación y gestión de órdenes
Estados: PENDING, CONFIRMED, CANCELLED
Publicación de eventos ecommerce.orders.placed
Consumo de eventos de confirmación/cancelación
Base de datos: ecommerce_orders

3. Inventory Service (Puerto 8082)

Gestión de inventario por producto
Control de stock disponible
Consumo de eventos ecommerce.orders.placed
Publicación de eventos de confirmación/cancelación
Base de datos: ecommerce_inventory

🚀 Requisitos Previos

Docker 20.x o superior
Docker Compose 2.x o superior
Maven 3.8+ (para compilación local)
Java 17+ (para compilación local)
Postman (para pruebas de API)
8GB RAM mínimo recomendado

📥 Instalación y Ejecución
1. Clonar el Repositorio
bashgit clone https://github.com/tu-usuario/ecommerce-kafka.git
cd ecommerce-kafka
2. Compilar los Microservicios
bash# Compilar Product Service
cd product-service
mvn clean install
cd ..

# Compilar Order Service
cd order-service
mvn clean install
cd ..

# Compilar Inventory Service
cd inventory-service
mvn clean install
cd ..
3. Levantar la Infraestructura con Docker
bash# Iniciar todos los servicios
docker-compose up -d

# Verificar que todos los contenedores estén corriendo
docker-compose ps
```

**Salida esperada:**
```
NAME                  STATUS    PORTS
postgres-products     Up        0.0.0.0:5435->5432/tcp
postgres-orders       Up        0.0.0.0:5436->5432/tcp
postgres-inventory    Up        0.0.0.0:5437->5432/tcp
kafka                 Up        0.0.0.0:9092->9092/tcp
product-service       Up        0.0.0.0:8080->8080/tcp
order-service         Up        0.0.0.0:8081->8081/tcp
inventory-service     Up        0.0.0.0:8082->8082/tcp
4. Verificar Topics de Kafka
bash# Listar topics
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --list

# Crear topics manualmente (si no se auto-crean)
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 \
  --create --topic ecommerce.products.created --partitions 5 --replication-factor 1

docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 \
  --create --topic ecommerce.orders.placed --partitions 5 --replication-factor 1

docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 \
  --create --topic ecommerce.orders.confirmed --partitions 5 --replication-factor 1

docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 \
  --create --topic ecommerce.orders.cancelled --partitions 5 --replication-factor 1
5. Verificar Logs de Servicios
bash# Ver logs de Product Service
docker-compose logs -f product-service

# Ver logs de Order Service
docker-compose logs -f order-service

# Ver logs de Inventory Service
docker-compose logs -f inventory-service
```

## 🔌 Endpoints de la API

### Product Service (http://localhost:8080)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/products` | Crear un nuevo producto |
| GET | `/api/products` | Listar todos los productos |
| GET | `/api/products/{id}` | Obtener producto por ID |
| PUT | `/api/products/{id}` | Actualizar producto |
| DELETE | `/api/products/{id}` | Eliminar producto |
| GET | `/api/products/category/{categoryId}` | Listar productos por categoría |
| POST | `/api/categories` | Crear categoría |
| GET | `/api/categories` | Listar categorías |

### Order Service (http://localhost:8081)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/orders` | Crear nueva orden |
| GET | `/api/orders` | Listar todas las órdenes |
| GET | `/api/orders/{id}` | Obtener orden por ID |
| GET | `/api/orders/status/{status}` | Listar órdenes por estado |
| PUT | `/api/orders/{id}/cancel` | Cancelar orden |

### Inventory Service (http://localhost:8082)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/inventory` | Crear registro de inventario |
| GET | `/api/inventory` | Listar todo el inventario |
| GET | `/api/inventory/{id}` | Obtener inventario por ID |
| GET | `/api/inventory/product/{productId}` | Consultar stock de producto |
| PUT | `/api/inventory/{id}` | Actualizar inventario |

## 🔄 Flujo de Eventos Kafka

### 1. Creación de Producto
```
Product Service → Produce → ecommerce.products.created
                           ↓
                    (Otros servicios pueden consumir)
```

### 2. Creación de Orden
```
Order Service → Produce → ecommerce.orders.placed
                         ↓
                  Inventory Service (Consume)
                         ↓
              ¿Hay stock suficiente?
                 /              \
              SÍ                NO
               ↓                 ↓
    ecommerce.orders.confirmed  ecommerce.orders.cancelled
               ↓                 ↓
       Order Service (Consume)  Order Service (Consume)
               ↓                 ↓
       Estado: CONFIRMED    Estado: CANCELLED
💾 Modelo de Datos
Product Service
Tabla: products
sqlid          BIGINT (PK)
name        VARCHAR(255) NOT NULL
description TEXT
price       DECIMAL(10,2) NOT NULL
category_id BIGINT (FK)
created_at  TIMESTAMP
updated_at  TIMESTAMP
Tabla: categories
sqlid          BIGINT (PK)
name        VARCHAR(100) NOT NULL
description TEXT
created_at  TIMESTAMP
Order Service
Tabla: orders
sqlid           BIGINT (PK)
product_id   BIGINT NOT NULL
quantity     INTEGER NOT NULL
total_amount DECIMAL(10,2) NOT NULL
status       VARCHAR(50) NOT NULL
customer_name VARCHAR(255)
created_at   TIMESTAMP
updated_at   TIMESTAMP
Inventory Service
Tabla: inventory
sqlid              BIGINT (PK)
product_id      BIGINT NOT NULL UNIQUE
available_stock INTEGER NOT NULL
reserved_stock  INTEGER DEFAULT 0
created_at      TIMESTAMP
updated_at      TIMESTAMP
🧪 Pruebas con Postman

Importar la colección desde postman/Ecommerce-Kafka.postman_collection.json
Configurar las variables de entorno:

product_service_url: http://localhost:8080
order_service_url: http://localhost:8081
inventory_service_url: http://localhost:8082



Flujo de Prueba Completo

Crear una categoría
Crear un producto (se genera evento en Kafka)
Crear inventario para el producto
Crear una orden (se valida stock automáticamente)
Verificar estado de la orden (debe ser CONFIRMED o CANCELLED)

🔧 Configuración
Variables de Entorno
Cada microservicio utiliza las siguientes variables (con valores por defecto):
yaml# Base de Datos
DB_HOST: localhost (docker: postgres-products/orders/inventory)
DB_PORT: 5432
DB_NAME: ecommerce_products / ecommerce_orders / ecommerce_inventory
DB_USER: product_user / order_user / inventory_user
DB_PASSWORD: product_password / order_password / inventory_password

# Kafka
KAFKA_BOOTSTRAP_SERVERS: localhost:9092 (docker: kafka:9092)

# Servidor
SERVER_PORT: 8080 / 8081 / 8082
Perfiles de Spring

default: Configuración local (localhost)
docker: Configuración para ejecución en contenedores
dev: Desarrollo con logs detallados
prod: Producción con logs optimizados

Activar un perfil:
bash# Local
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Docker (ya configurado en docker-compose.yml)
SPRING_PROFILES_ACTIVE=docker
📊 Evidencias de Funcionamiento
1. Servicios Levantados
Mostrar imagen
2. Topics de Kafka Creados
bash$ docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --list
ecommerce.orders.cancelled
ecommerce.orders.confirmed
ecommerce.orders.placed
ecommerce.products.created
3. Consumo de Mensajes en Kafka
bash# Monitorear eventos de órdenes
docker exec -it kafka kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic ecommerce.orders.placed \
  --from-beginning
Ejemplo de log:
json{
  "orderId": 1,
  "productId": 101,
  "quantity": 5,
  "status": "PENDING",
  "timestamp": "2025-11-27T10:30:00"
}
4. Ejemplo de Request/Response en Postman
Mostrar imagen
🐛 Troubleshooting
Los servicios no se conectan a Kafka
bash# Verificar que Kafka esté healthy
docker-compose ps kafka

# Revisar logs de Kafka
docker-compose logs kafka
Error de conexión a base de datos
bash# Verificar que PostgreSQL esté healthy
docker-compose ps | grep postgres

# Conectarse manualmente para verificar
docker exec -it postgres-products psql -U product_user -d ecommerce_products
Los topics no se crean automáticamente
bash# Crear manualmente (ver sección 4 de Instalación)
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 \
  --create --topic <topic-name> --partitions 5 --replication-factor 1
🛑 Detener el Sistema
bash# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes (CUIDADO: borra los datos)
docker-compose down -v
📚 Recursos Adicionales

Spring Boot Documentation
Spring Kafka Documentation
Apache Kafka Documentation
Docker Compose Documentation

👨‍💻 Autor
Tu Nombre

GitHub: @tu-usuario
Email: tu-email@ejemplo.com

📄 Licencia
Este proyecto es parte del curso de Microservicios y está bajo la Licencia MIT.

Nota: Este README debe ser actualizado con capturas de pantalla reales, logs específicos y el enlace correcto a tu repositorio antes de la entrega final.Ly podrias describir como se hizo el tema con docker compose donde se publico los servicios?Despliegue con Docker Compose
📦 Estrategia de Containerización
Este proyecto utiliza Docker Compose para orquestar todos los servicios del sistema de e-commerce. La estrategia implementada incluye:

Separación de bases de datos por microservicio
Configuración de Kafka en modo KRaft (sin Zookeeper)
Health checks para garantizar el orden de inicio
Variables de entorno para configuración dinámica
Red compartida para comunicación entre contenedores

🏗️ Arquitectura de Contenedores
┌─────────────────────────────────────────────────────────────┐
│                    ecommerce-network (bridge)               │
│                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐ │
│  │ postgres-      │  │ postgres-      │  │ postgres-    │ │
│  │ products       │  │ orders         │  │ inventory    │ │
│  │ :5435          │  │ :5436          │  │ :5437        │ │
│  └────────────────┘  └────────────────┘  └──────────────┘ │
│         ▲                   ▲                   ▲          │
│         │                   │                   │          │
│  ┌──────┴──────┐    ┌───────┴──────┐    ┌──────┴────────┐│
│  │ product-    │    │ order-       │    │ inventory-    ││
│  │ service     │◄───┤ service      │◄───┤ service       ││
│  │ :8080       │    │ :8081        │    │ :8082         ││
│  └──────┬──────┘    └───────┬──────┘    └──────┬────────┘│
│         │                   │                   │          │
│         └───────────────────┼───────────────────┘          │
│                             ▼                              │
│                      ┌─────────────┐                       │
│                      │   Kafka     │                       │
│                      │   :9092     │                       │
│                      └─────────────┘                       │
└─────────────────────────────────────────────────────────────┘
📝 Estructura del Proyecto
proyectoKafka/
├── docker-compose.yml              # Orquestación de servicios
├── product-service/
│   ├── Dockerfile                  # Imagen del servicio
│   ├── pom.xml
│   ├── src/
│   └── target/
│       └── product-service-1.0.jar # JAR compilado
├── order-service/
│   ├── Dockerfile
│   ├── pom.xml
│   ├── src/
│   └── target/
│       └── order-service-0.0.1-SNAPSHOT.jar
└── inventory-service/
    ├── Dockerfile
    ├── pom.xml
    ├── src/
    └── target/
        └── inventoryservice-0.0.1-SNAPSHOT.jar
🐳 Dockerfiles de los Microservicios
Cada microservicio tiene su propio Dockerfile optimizado:
Estructura del Dockerfile (ejemplo genérico)
dockerfile# Usa una imagen base ligera de Java
FROM eclipse-temurin:17-jre-alpine

# Establece el directorio de trabajo
WORKDIR /app

# Copia el JAR compilado
COPY target/*.jar app.jar

# Expone el puerto del servicio
EXPOSE 8080

# Comando para ejecutar la aplicación
ENTRYPOINT ["java", "-jar", "app.jar"]
Product Service Dockerfile
dockerfileFROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/product-service-1.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
Order Service Dockerfile
dockerfileFROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/order-service-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
Inventory Service Dockerfile
dockerfileFROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/inventoryservice-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]
🔧 Análisis del docker-compose.yml
1. Bases de Datos PostgreSQL
Cada microservicio tiene su propia base de datos para mantener la independencia de datos:
yamlpostgres-products:
  image: postgres:15-alpine          # Imagen oficial ligera
  container_name: postgres-products
  restart: unless-stopped            # Reinicio automático
  environment:
    POSTGRES_DB: ecommerce_products  # Nombre de la BD
    POSTGRES_USER: product_user      # Usuario específico
    POSTGRES_PASSWORD: product_password
  ports:
    - "5435:5432"                    # Puerto externo:interno
  volumes:
    - postgres-products-data:/var/lib/postgresql/data  # Persistencia
  networks:
    - ecommerce-network
  healthcheck:                       # Verificación de salud
    test: ["CMD-SHELL", "pg_isready -U product_user -d ecommerce_products"]
    interval: 10s
    timeout: 5s
    retries: 5
Puntos clave:

Puertos mapeados: 5435:5432, 5436:5432, 5437:5432 (evita conflictos)
Volúmenes nombrados: Datos persistentes incluso si se elimina el contenedor
Health checks: Garantiza que la BD esté lista antes de iniciar los servicios

2. Apache Kafka (Modo KRaft)
Kafka se configura sin Zookeeper usando el nuevo modo KRaft:
yamlkafka:
  image: confluentinc/cp-kafka:latest
  container_name: kafka
  restart: unless-stopped
  ports:
    - "9092:9092"                    # Puerto para clientes externos
    - "9093:9093"                    # Puerto para controlador
  environment:
    KAFKA_NODE_ID: 1
    KAFKA_PROCESS_ROLES: broker,controller  # Modo KRaft
    KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092  # Nombre del servicio
    KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
    KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
    KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
    KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"  # Crea topics automáticamente
    CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qk"
  volumes:
    - kafka-data:/var/lib/kafka/data
  networks:
    - ecommerce-network
  healthcheck:
    test: ["CMD-SHELL", "kafka-broker-api-versions --bootstrap-server localhost:9092"]
    interval: 10s
    timeout: 10s
    retries: 5
Configuración destacada:

KAFKA_ADVERTISED_LISTENERS: kafka:9092 permite que los servicios se conecten usando el nombre del contenedor
KAFKA_AUTO_CREATE_TOPICS_ENABLE: Topics se crean automáticamente al primer uso
Factor de replicación = 1: Adecuado para desarrollo (en producción sería mayor)

3. Microservicios Spring Boot
Cada servicio se construye desde su contexto local y se conecta a su BD:
yamlproduct-service:
  build:
    context: ./product-service       # Carpeta con Dockerfile
    dockerfile: Dockerfile
  container_name: product-service
  restart: unless-stopped
  ports:
    - "8080:8080"                    # API REST expuesta
  environment:
    SPRING_PROFILES_ACTIVE: docker   # Perfil específico para Docker
    DB_HOST: postgres-products       # Nombre del servicio de BD
    DB_PORT: 5432                    # Puerto interno
    DB_NAME: ecommerce_products
    DB_USER: product_user
    DB_PASSWORD: product_password
    KAFKA_BOOTSTRAP_SERVERS: kafka:9092  # Kafka interno
    SERVER_PORT: 8080
  depends_on:                        # Orden de inicio
    postgres-products:
      condition: service_healthy     # Espera hasta que esté healthy
    kafka:
      condition: service_healthy
  networks:
    - ecommerce-network
Dependencias (depends_on):

Garantiza que PostgreSQL y Kafka estén completamente listos antes de iniciar
condition: service_healthy espera el health check exitoso

4. Volúmenes y Redes
yamlvolumes:
  postgres-products-data:   # Persistencia de datos
  postgres-orders-data:
  postgres-inventory-data:
  kafka-data:

networks:
  ecommerce-network:        # Red bridge compartida
    driver: bridge
Beneficios:

Volúmenes nombrados: Los datos sobreviven a docker-compose down
Red bridge: Todos los servicios se comunican por nombre (DNS interno)

🚀 Proceso de Construcción y Despliegue
Paso 1: Compilar los JARs
Antes de levantar Docker, cada microservicio debe estar compilado:
bash# Product Service
cd product-service
mvn clean package -DskipTests
cd ..

# Order Service
cd order-service
mvn clean package -DskipTests
cd ..

# Inventory Service
cd inventory-service
mvn clean package -DskipTests
cd ..
```

**Resultado:**
```
product-service/target/product-service-1.0.jar
order-service/target/order-service-0.0.1-SNAPSHOT.jar
inventory-service/target/inventoryservice-0.0.1-SNAPSHOT.jar
Paso 2: Construir las Imágenes Docker
bash# Construir todas las imágenes
docker-compose build

# O construir individualmente
docker-compose build product-service
docker-compose build order-service
docker-compose build inventory-service
Lo que sucede:

Docker lee cada Dockerfile
Copia el JAR al contenedor
Crea una imagen optimizada con Java 17
Etiqueta la imagen (ej: proyectokafka-product-service)

Paso 3: Levantar los Servicios
bash# Iniciar todos los servicios en modo detached
docker-compose up -d
Orden de inicio (gracias a depends_on y health checks):

PostgreSQL (productos, órdenes, inventario)
Kafka (broker + controlador)
Product Service (cuando postgres-products y kafka estén healthy)
Order Service (cuando postgres-orders y kafka estén healthy)
Inventory Service (cuando postgres-inventory y kafka estén healthy)

Paso 4: Verificar el Despliegue
bash# Ver estado de todos los contenedores
docker-compose ps

# Salida esperada:
NAME                  IMAGE                              STATUS    PORTS
inventory-service     proyectokafka-inventory-service    Up        0.0.0.0:8082->8082/tcp
kafka                 confluentinc/cp-kafka:latest       Up        0.0.0.0:9092-9093->9092-9093/tcp
order-service         proyectokafka-order-service        Up        0.0.0.0:8081->8081/tcp
postgres-inventory    postgres:15-alpine                 Up        0.0.0.0:5437->5432/tcp
postgres-orders       postgres:15-alpine                 Up        0.0.0.0:5436->5432/tcp
postgres-products     postgres:15-alpine                 Up        0.0.0.0:5435->5432/tcp
product-service       proyectokafka-product-service      Up        0.0.0.0:8080->8080/tcp
🔍 Verificación de Funcionamiento
1. Verificar Conectividad de Red
bash# Entrar a un contenedor y hacer ping a otro
docker exec -it product-service ping kafka -c 3

# Salida esperada:
PING kafka (172.18.0.X): 56 data bytes
64 bytes from 172.18.0.X: seq=0 ttl=64 time=0.123 ms
2. Verificar Bases de Datos
bash# Conectarse a PostgreSQL de productos
docker exec -it postgres-products psql -U product_user -d ecommerce_products

# Dentro de psql:
\dt           # Listar tablas
\q            # Salir
3. Verificar Topics de Kafka
bash# Listar topics creados
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --list

# Salida esperada:
ecommerce.orders.cancelled
ecommerce.orders.confirmed
ecommerce.orders.placed
ecommerce.products.created
4. Verificar Logs de Microservicios
bash# Ver logs en tiempo real
docker-compose logs -f product-service

# Salida esperada:
product-service  | Started ProductServiceApplication in 12.345 seconds
product-service  | Tomcat started on port(s): 8080 (http)
5. Probar los Endpoints
bash# Health check del Product Service
curl http://localhost:8080/actuator/health

# Salida esperada:
{"status":"UP"}

# Listar productos
curl http://localhost:8080/api/products
🔄 Flujo de Comunicación entre Servicios
Escenario: Crear una Orden

Request HTTP → Order Service (localhost:8081)

bash   curl -X POST http://localhost:8081/api/orders \
     -H "Content-Type: application/json" \
     -d '{
       "productId": 1,
       "quantity": 5,
       "customerName": "Juan Pérez"
     }'
```

2. **Order Service** → Publica evento en Kafka
```
   Topic: ecommerce.orders.placed
   Mensaje: {orderId: 1, productId: 1, quantity: 5, status: "PENDING"}
```

3. **Inventory Service** ← Consume evento de Kafka
```
   - Verifica stock disponible
   - Si hay stock: Reserva y publica "ecommerce.orders.confirmed"
   - Si no hay: Publica "ecommerce.orders.cancelled"
```

4. **Order Service** ← Consume respuesta
```
   - Actualiza estado de la orden a CONFIRMED o CANCELLED
```

### Comunicación a través de Docker Network
```
┌─────────────────┐
│ Order Service   │
│ Container       │
└────────┬────────┘
         │ kafka:9092 (DNS interno)
         ▼
┌─────────────────┐
│ Kafka           │
│ Container       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Inventory Svc   │
│ Container       │
└─────────────────┘
Ventaja de Docker Network:

Los servicios usan kafka:9092 en lugar de localhost:9092
Docker resuelve automáticamente el nombre al IP del contenedor

🛠️ Comandos Útiles de Gestión
Ver Logs Específicos
bash# Logs de un servicio específico
docker-compose logs product-service

# Últimas 50 líneas
docker-compose logs --tail=50 order-service

# Tiempo real con filtro
docker-compose logs -f kafka | grep ERROR
Reiniciar Servicios
bash# Reiniciar un servicio específico
docker-compose restart product-service

# Reiniciar todo
docker-compose restart
Reconstruir Imágenes
bash# Reconstruir tras cambios en código
docker-compose build --no-cache product-service
docker-compose up -d product-service
Escalar Servicios (si aplica)
bash# Levantar 3 instancias del product-service
docker-compose up -d --scale product-service=3
Limpiar Recursos
bash# Detener y eliminar contenedores
docker-compose down

# Eliminar también volúmenes (¡CUIDADO! Borra datos)
docker-compose down -v

# Eliminar imágenes huérfanas
docker image prune -f
📊 Monitoreo y Debugging
Ver Uso de Recursos
bash# Estadísticas en tiempo real
docker stats

# Salida:
CONTAINER           CPU %    MEM USAGE / LIMIT     MEM %    NET I/O
product-service     0.50%    512MiB / 2GiB        25.60%   1.2MB / 890kB
kafka               2.30%    1.2GiB / 2GiB        60.00%   5.4MB / 3.2MB
postgres-products   0.10%    64MiB / 2GiB         3.20%    450kB / 280kB
Inspeccionar Configuración
bash# Ver configuración de red
docker network inspect proyectokafka_ecommerce-network

# Ver configuración de volumen
docker volume inspect proyectokafka_postgres-products-data

# Ver variables de entorno de un contenedor
docker exec product-service env
Ejecutar Comandos Dentro del Contenedor
bash# Abrir shell en el contenedor
docker exec -it product-service sh

# Verificar conectividad
docker exec product-service ping kafka -c 3

# Ver procesos
docker exec product-service ps aux
⚠️ Troubleshooting Común
Problema 1: Servicios no inician
bash# Verificar health checks
docker-compose ps

# Si postgres está "unhealthy":
docker-compose logs postgres-products

# Solución: Esperar más tiempo o ajustar health check interval
Problema 2: Kafka no se conecta
bash# Verificar configuración de Kafka
docker exec kafka env | grep KAFKA

# Probar conectividad desde otro contenedor
docker exec product-service nc -zv kafka 9092
Problema 3: Base de datos no accesible
bash# Verificar que el puerto esté expuesto
docker-compose ps | grep postgres

# Probar conexión externa
psql -h localhost -p 5435 -U product_user -d ecommerce_products
Problema 4: Cambios en código no se reflejan
bash# Recompilar JAR
cd product-service
mvn clean package -DskipTests
cd ..

# Reconstruir imagen
docker-compose build --no-cache product-service

# Recrear contenedor
docker-compose up -d --force-recreate product-service
🎯 Mejores Prácticas Aplicadas

Health Checks: Garantizan inicio ordenado y detectan fallos
Volúmenes Nombrados: Datos persistentes y fáciles de respaldar
Red Personalizada: Aislamiento y comunicación por nombre
Variables de Entorno: Configuración flexible sin cambiar código
Restart Policies: Alta disponibilidad ante fallos
Perfiles de Spring: Configuración específica para Docker
Puerto Mapping: Evita conflictos con múltiples PostgreSQL

📈 Escalabilidad Futura
Este docker-compose puede extenderse fácilmente:
yaml# Agregar réplicas de Kafka
kafka-2:
  image: confluentinc/cp-kafka:latest
  # ... configuración similar

# Agregar load balancer
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
  depends_on:
    - product-service
    - order-service
    - inventory-service

# Agregar monitoreo
prometheus:
  image: prom/prometheus
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml

Resumen: La estrategia con Docker Compose proporciona un entorno reproducible, escalable y fácil de gestionar, ideal para desarrollo y pruebas del sistema de e-commerce distribuido.
