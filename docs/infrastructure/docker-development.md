# 🐳 Docker Development Environment

Guía completa para configurar un entorno de desarrollo local optimizado con Docker Compose, incluyendo hot-reload, SSL automático con Traefik, y observabilidad.

## 🎯 ¿Por qué Docker para Desarrollo?

En ecommerce, el entorno local debe **replicar producción**:

- **🔄 Consistencia**: Mismo comportamiento en todos los entornos
- **⚡ Setup rápido**: Nuevos developers productivos en minutos
- **🔌 Servicios completos**: PostgreSQL, Redis, workers, monitoring
- **🌐 HTTPS local**: SSL automático para testing real
- **📊 Observabilidad**: Métricas y logs desde día 1

### Beneficios del Stack Docker

- **🚀 Time-to-productivity**: < 15 minutos desde git clone
- **🔄 Hot-reload**: Cambios instantáneos sin rebuild
- **🌐 Networking**: Comunicación entre servicios automática
- **📦 Isolation**: Cada servicio en su contenedor
- **🎛️ Orchestration**: Traefik como proxy inteligente

## 🏗️ Arquitectura Local

### Stack de Desarrollo

```mermaid
graph TB
    subgraph "🌐 Edge Layer"
        TRAEFIK[Traefik Proxy :80/:443]
        LE[Let's Encrypt]
    end

    subgraph "🎨 Frontend Layer"
        ANGULAR[Angular App :4200]
        ADMIN[Admin Panel :4201]
    end

    subgraph "⚙️ Backend Layer"
        GATEWAY[API Gateway :3000]
        AUTH[Auth Service :3001]
        PRODUCTS[Products Service :3002]
        ORDERS[Orders Service :3003]
        PAYMENTS[Payments Service :3004]
        NOTIFICATIONS[Notifications Service :3005]
    end

    subgraph "💾 Data Layer"
        POSTGRES[(PostgreSQL :5432)]
        REDIS[(Redis :6379)]
        PGADMIN[PgAdmin :5050]
    end

    subgraph "📊 Observability Layer"
        PROMETHEUS[Prometheus :9090]
        GRAFANA[Grafana :3001]
        LOKI[Loki :3100]
        JAEGER[Jaeger :16686]
    end

    TRAEFIK --> ANGULAR
    TRAEFIK --> ADMIN
    TRAEFIK --> GATEWAY
    TRAEFIK --> PGADMIN
    TRAEFIK --> GRAFANA
    TRAEFIK --> PROMETHEUS
    TRAEFIK --> JAEGER

    GATEWAY --> AUTH
    GATEWAY --> PRODUCTS
    GATEWAY --> ORDERS
    GATEWAY --> PAYMENTS
    GATEWAY --> NOTIFICATIONS

    AUTH --> REDIS
    PRODUCTS --> POSTGRES
    ORDERS --> POSTGRES
    PAYMENTS --> POSTGRES
    NOTIFICATIONS --> POSTGRES

    LE -.-> TRAEFIK

    style TRAEFIK fill:#f39c12,color:#fff
    style POSTGRES fill:#336791,color:#fff
    style REDIS fill:#dc382d,color:#fff
    style PROMETHEUS fill:#e6522c,color:#fff
    style GRAFANA fill:#f46800,color:#fff
```

## 📁 Estructura de Archivos Docker

```
my-ecommerce-project/
├── docker-compose.yml              # Configuración principal
├── docker-compose.override.yml     # Overrides para desarrollo
├── docker-compose.prod.yml         # Configuración de producción
├── .env                           # Variables de entorno
├── .env.example                   # Template de variables
│
├── docker/                        # Configuraciones Docker
│   ├── traefik/
│   │   ├── traefik.yml           # Configuración Traefik
│   │   ├── dynamic.yml           # Configuración dinámica
│   │   └── certs/                # Certificados locales
│   │
│   ├── postgres/
│   │   ├── init.sql              # Script de inicialización
│   │   └── postgresql.conf       # Configuración PostgreSQL
│   │
│   ├── monitoring/
│   │   ├── prometheus.yml        # Configuración Prometheus
│   │   ├── grafana/
│   │   │   ├── dashboards/       # Dashboards predefinidos
│   │   │   └── datasources/      # Fuentes de datos
│   │   └── loki/
│   │       └── loki-config.yml   # Configuración Loki
│   │
│   └── nginx/                    # Nginx para assets estáticos
│       └── nginx.conf
│
├── apps/                         # Aplicaciones del monorepo
│   ├── frontend/
│   │   └── Dockerfile.dev        # Dockerfile para desarrollo
│   ├── api-gateway/
│   │   └── Dockerfile.dev
│   └── auth-service/
│       └── Dockerfile.dev
│
└── scripts/                      # Scripts de automatización
    ├── setup-dev.sh             # Setup inicial
    ├── reset-db.sh              # Reset de base de datos
    └── health-check.sh           # Verificación de servicios
```

## 🐳 Configuración Docker Compose

### docker-compose.yml Principal

```yaml
# docker-compose.yml
version: "3.8"

services:
  # ===================
  # Proxy & Load Balancer
  # ===================
  traefik:
    image: traefik:v3.0
    container_name: ecommerce-traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080" # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./docker/traefik/traefik.yml:/traefik.yml:ro
      - ./docker/traefik/dynamic.yml:/dynamic.yml:ro
      - ./docker/traefik/certs:/certs
      - traefik-acme:/acme
    environment:
      - TRAEFIK_API_DASHBOARD=true
      - TRAEFIK_API_INSECURE=true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`traefik.${DOMAIN}`)"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.tls.certresolver=letsencrypt"
    networks:
      - ecommerce-network

  # ===================
  # Frontend Applications
  # ===================
  frontend:
    build:
      context: .
      dockerfile: apps/frontend/Dockerfile.dev
    container_name: ecommerce-frontend
    restart: unless-stopped
    volumes:
      - ./apps/frontend:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - API_URL=https://api.${DOMAIN}
      - CHOKIDAR_USEPOLLING=true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`${DOMAIN}`, `www.${DOMAIN}`)"
      - "traefik.http.routers.frontend.tls=true"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
      - "traefik.http.services.frontend.loadbalancer.server.port=4200"
    depends_on:
      - api-gateway
    networks:
      - ecommerce-network

  admin:
    build:
      context: .
      dockerfile: apps/admin/Dockerfile.dev
    container_name: ecommerce-admin
    restart: unless-stopped
    volumes:
      - ./apps/admin:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - API_URL=https://api.${DOMAIN}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.admin.rule=Host(`admin.${DOMAIN}`)"
      - "traefik.http.routers.admin.tls=true"
      - "traefik.http.routers.admin.tls.certresolver=letsencrypt"
      - "traefik.http.services.admin.loadbalancer.server.port=4201"
    networks:
      - ecommerce-network

  # ===================
  # Backend Services
  # ===================
  api-gateway:
    build:
      context: .
      dockerfile: apps/api-gateway/Dockerfile.dev
    container_name: ecommerce-api-gateway
    restart: unless-stopped
    volumes:
      - ./apps/api-gateway:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3000
      - REDIS_URL=redis://redis:6379
      - AUTH_SERVICE_URL=http://auth-service:3001
      - PRODUCTS_SERVICE_URL=http://products-service:3002
      - ORDERS_SERVICE_URL=http://orders-service:3003
      - PAYMENTS_SERVICE_URL=http://payments-service:3004
      - NOTIFICATIONS_SERVICE_URL=http://notifications-service:3005
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api-gateway.rule=Host(`api.${DOMAIN}`)"
      - "traefik.http.routers.api-gateway.tls=true"
      - "traefik.http.routers.api-gateway.tls.certresolver=letsencrypt"
      - "traefik.http.services.api-gateway.loadbalancer.server.port=3000"
    depends_on:
      - redis
      - postgres
    networks:
      - ecommerce-network

  auth-service:
    build:
      context: .
      dockerfile: apps/auth-service/Dockerfile.dev
    container_name: ecommerce-auth-service
    restart: unless-stopped
    volumes:
      - ./apps/auth-service:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3001
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/auth_db
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce-network

  products-service:
    build:
      context: .
      dockerfile: apps/products-service/Dockerfile.dev
    container_name: ecommerce-products-service
    restart: unless-stopped
    volumes:
      - ./apps/products-service:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3002
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/products_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce-network

  orders-service:
    build:
      context: .
      dockerfile: apps/orders-service/Dockerfile.dev
    container_name: ecommerce-orders-service
    restart: unless-stopped
    volumes:
      - ./apps/orders-service:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3003
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/orders_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce-network

  payments-service:
    build:
      context: .
      dockerfile: apps/payments-service/Dockerfile.dev
    container_name: ecommerce-payments-service
    restart: unless-stopped
    volumes:
      - ./apps/payments-service:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3004
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/payments_db
      - REDIS_URL=redis://redis:6379
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
      - STRIPE_WEBHOOK_SECRET=${STRIPE_WEBHOOK_SECRET}
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce-network

  notifications-service:
    build:
      context: .
      dockerfile: apps/notifications-service/Dockerfile.dev
    container_name: ecommerce-notifications-service
    restart: unless-stopped
    volumes:
      - ./apps/notifications-service:/app
      - /app/node_modules
      - /app/dist
    environment:
      - NODE_ENV=development
      - PORT=3005
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/notifications_db
      - REDIS_URL=redis://redis:6379
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - SMTP_USER=${SMTP_USER}
      - SMTP_PASS=${SMTP_PASS}
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce-network

  # ===================
  # Database Services
  # ===================
  postgres:
    image: postgres:15-alpine
    container_name: ecommerce-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=ecommerce
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./docker/postgres/postgresql.conf:/etc/postgresql/postgresql.conf
    ports:
      - "5432:5432"
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    networks:
      - ecommerce-network

  redis:
    image: redis:7-alpine
    container_name: ecommerce-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"
    networks:
      - ecommerce-network

  # ===================
  # Database Management
  # ===================
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: ecommerce-pgadmin
    restart: unless-stopped
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_EMAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_PASSWORD}
      - PGADMIN_CONFIG_SERVER_MODE=False
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.pgadmin.rule=Host(`pgadmin.${DOMAIN}`)"
      - "traefik.http.routers.pgadmin.tls=true"
      - "traefik.http.routers.pgadmin.tls.certresolver=letsencrypt"
      - "traefik.http.services.pgadmin.loadbalancer.server.port=80"
    depends_on:
      - postgres
    networks:
      - ecommerce-network

  # ===================
  # Monitoring Stack
  # ===================
  prometheus:
    image: prom/prometheus:latest
    container_name: ecommerce-prometheus
    restart: unless-stopped
    volumes:
      - ./docker/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
      - "--web.console.libraries=/etc/prometheus/console_libraries"
      - "--web.console.templates=/etc/prometheus/consoles"
      - "--storage.tsdb.retention.time=200h"
      - "--web.enable-lifecycle"
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.prometheus.rule=Host(`prometheus.${DOMAIN}`)"
      - "traefik.http.routers.prometheus.tls=true"
      - "traefik.http.routers.prometheus.tls.certresolver=letsencrypt"
      - "traefik.http.services.prometheus.loadbalancer.server.port=9090"
    networks:
      - ecommerce-network

  grafana:
    image: grafana/grafana:latest
    container_name: ecommerce-grafana
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./docker/monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./docker/monitoring/grafana/datasources:/etc/grafana/provisioning/datasources
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.grafana.rule=Host(`grafana.${DOMAIN}`)"
      - "traefik.http.routers.grafana.tls=true"
      - "traefik.http.routers.grafana.tls.certresolver=letsencrypt"
      - "traefik.http.services.grafana.loadbalancer.server.port=3000"
    depends_on:
      - prometheus
    networks:
      - ecommerce-network

  loki:
    image: grafana/loki:latest
    container_name: ecommerce-loki
    restart: unless-stopped
    volumes:
      - ./docker/monitoring/loki/loki-config.yml:/etc/loki/local-config.yaml
      - loki-data:/tmp/loki
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - ecommerce-network

  jaeger:
    image: jaegertracing/all-in-one:latest
    container_name: ecommerce-jaeger
    restart: unless-stopped
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.jaeger.rule=Host(`jaeger.${DOMAIN}`)"
      - "traefik.http.routers.jaeger.tls=true"
      - "traefik.http.routers.jaeger.tls.certresolver=letsencrypt"
      - "traefik.http.services.jaeger.loadbalancer.server.port=16686"
    networks:
      - ecommerce-network

# ===================
# Volumes
# ===================
volumes:
  postgres-data:
  redis-data:
  pgadmin-data:
  prometheus-data:
  grafana-data:
  loki-data:
  traefik-acme:

# ===================
# Networks
# ===================
networks:
  ecommerce-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### .env Template

```bash
# .env.example
# ===================
# Domain Configuration
# ===================
DOMAIN=dev.midominio.com

# ===================
# Database Configuration
# ===================
POSTGRES_USER=ecommerce_user
POSTGRES_PASSWORD=strong_password_here
POSTGRES_DB=ecommerce

REDIS_PASSWORD=redis_password_here

# ===================
# Auth Configuration
# ===================
JWT_SECRET=jwt_super_secret_key_minimum_32_chars
JWT_REFRESH_SECRET=jwt_refresh_super_secret_key_minimum_32_chars

# ===================
# Database Management
# ===================
PGADMIN_EMAIL=admin@midominio.com
PGADMIN_PASSWORD=pgadmin_password

# ===================
# Monitoring
# ===================
GRAFANA_USER=admin
GRAFANA_PASSWORD=grafana_password

# ===================
# Email Configuration
# ===================
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@midominio.com
SMTP_PASS=email_password

# ===================
# Payment Configuration
# ===================
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret

# ===================
# External Services
# ===================
CLOUDFLARE_API_TOKEN=your_cloudflare_token
```

## 🔧 Dockerfiles de Desarrollo

### Frontend Dockerfile

```dockerfile
# apps/frontend/Dockerfile.dev
FROM node:18-alpine

WORKDIR /app

# Instalar dependencias globales
RUN npm install -g @angular/cli@latest @nrwl/cli@latest

# Copiar package files
COPY package*.json ./
COPY nx.json ./
COPY tsconfig.base.json ./

# Instalar dependencias
RUN npm ci

# Copiar código fuente
COPY . .

# Exponer puerto
EXPOSE 4200

# Comando de desarrollo con hot-reload
CMD ["npx", "nx", "serve", "frontend", "--host", "0.0.0.0", "--port", "4200", "--poll", "2000"]
```

### Backend Service Dockerfile

```dockerfile
# apps/auth-service/Dockerfile.dev
FROM node:18-alpine

WORKDIR /app

# Instalar dependencias para development
RUN npm install -g @nestjs/cli@latest @nrwl/cli@latest nodemon

# Copiar package files
COPY package*.json ./
COPY nx.json ./
COPY tsconfig.base.json ./

# Instalar dependencias
RUN npm ci

# Copiar código fuente
COPY . .

# Exponer puerto
EXPOSE 3001

# Comando de desarrollo con hot-reload
CMD ["npx", "nx", "serve", "auth-service", "--host", "0.0.0.0", "--port", "3001"]
```

## ⚙️ Configuración de Traefik

### traefik.yml

```yaml
# docker/traefik/traefik.yml
api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entrypoint:
          to: websecure
          scheme: https
          permanent: true
  websecure:
    address: ":443"

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: ecommerce-network
  file:
    filename: /dynamic.yml
    watch: true

certificatesResolvers:
  letsencrypt:
    acme:
      tlsChallenge: {}
      email: admin@midominio.com
      storage: /acme/acme.json
      httpChallenge:
        entryPoint: web

# Logging
log:
  level: INFO
  format: json

accessLog:
  format: json
  bufferingSize: 100

# Metrics
metrics:
  prometheus:
    addEntryPointsLabels: true
    addServicesLabels: true
    addRoutersLabels: true
```

### dynamic.yml

```yaml
# docker/traefik/dynamic.yml
http:
  middlewares:
    default-headers:
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000
        customFrameOptionsValue: SAMEORIGIN
        referrerPolicy: "same-origin"

    secure-headers:
      headers:
        accessControlAllowMethods:
          - GET
          - OPTIONS
          - PUT
        accessControlAllowOriginList:
          - "https://dev.midominio.com"
          - "https://admin.dev.midominio.com"
        accessControlMaxAge: 100
        addVaryHeader: true

    rate-limit:
      rateLimit:
        average: 100
        burst: 50

tls:
  options:
    default:
      sslStrategies:
        - "tls.SniStrict"
      minVersion: "VersionTLS12"
      cipherSuites:
        - "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
        - "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305"
        - "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
        - "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256"
```

## 🗄️ Configuración de Base de Datos

### PostgreSQL Init Script

```sql
-- docker/postgres/init.sql
-- Crear bases de datos para cada servicio
CREATE DATABASE auth_db;
CREATE DATABASE products_db;
CREATE DATABASE orders_db;
CREATE DATABASE payments_db;
CREATE DATABASE notifications_db;

-- Crear usuario específico para la aplicación
CREATE USER ecommerce_app WITH PASSWORD 'app_specific_password';

-- Otorgar permisos
GRANT ALL PRIVILEGES ON DATABASE auth_db TO ecommerce_app;
GRANT ALL PRIVILEGES ON DATABASE products_db TO ecommerce_app;
GRANT ALL PRIVILEGES ON DATABASE orders_db TO ecommerce_app;
GRANT ALL PRIVILEGES ON DATABASE payments_db TO ecommerce_app;
GRANT ALL PRIVILEGES ON DATABASE notifications_db TO ecommerce_app;

-- Extensiones útiles
\c auth_db;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

\c products_db;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "unaccent";

\c orders_db;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

\c payments_db;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

\c notifications_db;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

### PostgreSQL Configuration

```conf
# docker/postgres/postgresql.conf
# Configuración optimizada para desarrollo

# Connection Settings
listen_addresses = '*'
port = 5432
max_connections = 100

# Memory Settings
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100

# Logging
logging_collector = on
log_directory = '/var/log/postgresql'
log_filename = 'postgresql-%Y-%m-%d.log'
log_statement = 'all'
log_min_duration_statement = 1000  # Log slow queries (1 second)

# Performance
random_page_cost = 1.1  # Para SSDs
effective_io_concurrency = 200
```

## 📊 Configuración de Monitoreo

### Prometheus Configuration

```yaml
# docker/monitoring/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "first_rules.yml"

scrape_configs:
  # Prometheus self-monitoring
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  # Traefik metrics
  - job_name: "traefik"
    static_configs:
      - targets: ["traefik:8080"]

  # Application services
  - job_name: "api-gateway"
    static_configs:
      - targets: ["api-gateway:3000"]
    metrics_path: "/metrics"

  - job_name: "auth-service"
    static_configs:
      - targets: ["auth-service:3001"]
    metrics_path: "/metrics"

  - job_name: "products-service"
    static_configs:
      - targets: ["products-service:3002"]
    metrics_path: "/metrics"

  - job_name: "orders-service"
    static_configs:
      - targets: ["orders-service:3003"]
    metrics_path: "/metrics"

  - job_name: "payments-service"
    static_configs:
      - targets: ["payments-service:3004"]
    metrics_path: "/metrics"

  - job_name: "notifications-service"
    static_configs:
      - targets: ["notifications-service:3005"]
    metrics_path: "/metrics"

  # Infrastructure monitoring
  - job_name: "postgres-exporter"
    static_configs:
      - targets: ["postgres-exporter:9187"]

  - job_name: "redis-exporter"
    static_configs:
      - targets: ["redis-exporter:9121"]

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093
```

### Grafana Datasource

```yaml
# docker/monitoring/grafana/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    editable: true
```

## 🚀 Scripts de Automatización

### Setup Script

```bash
#!/bin/bash
# scripts/setup-dev.sh

set -e

echo "🚀 Setting up ecommerce development environment..."

# Verificar dependencias
command -v docker >/dev/null 2>&1 || { echo "❌ Docker is required but not installed."; exit 1; }
command -v docker-compose >/dev/null 2>&1 || { echo "❌ Docker Compose is required but not installed."; exit 1; }

# Crear archivo .env si no existe
if [ ! -f .env ]; then
    echo "📄 Creating .env file from template..."
    cp .env.example .env
    echo "✅ Please edit .env file with your specific configuration"
fi

# Crear directorios necesarios
echo "📁 Creating necessary directories..."
mkdir -p docker/traefik/certs
mkdir -p docker/monitoring/grafana/dashboards
mkdir -p docker/monitoring/grafana/datasources
mkdir -p data/postgres
mkdir -p data/redis

# Generar certificados SSL locales para desarrollo
echo "🔐 Generating local SSL certificates..."
if [ ! -f docker/traefik/certs/local.crt ]; then
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout docker/traefik/certs/local.key \
        -out docker/traefik/certs/local.crt \
        -subj "/C=US/ST=Dev/L=Dev/O=Dev/CN=*.dev.midominio.com"
fi

# Construir imágenes
echo "🏗️ Building Docker images..."
docker-compose build

# Iniciar servicios base
echo "🚀 Starting base services..."
docker-compose up -d postgres redis

# Esperar a que la base de datos esté lista
echo "⏳ Waiting for database to be ready..."
while ! docker-compose exec postgres pg_isready -U ecommerce_user -d ecommerce >/dev/null 2>&1; do
    sleep 1
done

# Ejecutar migraciones (si existen)
echo "🔄 Running database migrations..."
# docker-compose exec api-gateway npm run migration:run

# Iniciar todos los servicios
echo "🌐 Starting all services..."
docker-compose up -d

# Verificar que todos los servicios estén funcionando
echo "🔍 Checking service health..."
sleep 10
./scripts/health-check.sh

echo "✅ Development environment setup complete!"
echo ""
echo "🌐 Available services:"
echo "  • Frontend: https://dev.midominio.com"
echo "  • Admin: https://admin.dev.midominio.com"
echo "  • API Gateway: https://api.dev.midominio.com"
echo "  • PgAdmin: https://pgadmin.dev.midominio.com"
echo "  • Grafana: https://grafana.dev.midominio.com"
echo "  • Prometheus: https://prometheus.dev.midominio.com"
echo "  • Jaeger: https://jaeger.dev.midominio.com"
echo "  • Traefik Dashboard: http://localhost:8080"
echo ""
echo "📝 Next steps:"
echo "  1. Edit .env file with your configuration"
echo "  2. Update your /etc/hosts file to point *.dev.midominio.com to localhost"
echo "  3. Visit https://dev.midominio.com to see your app"
```

### Health Check Script

```bash
#!/bin/bash
# scripts/health-check.sh

set -e

echo "🔍 Checking service health..."

# Function to check HTTP endpoint
check_http() {
    local name=$1
    local url=$2
    local expected_status=${3:-200}

    echo -n "  • $name: "
    if curl -s -o /dev/null -w "%{http_code}" "$url" | grep -q "$expected_status"; then
        echo "✅ OK"
    else
        echo "❌ FAILED"
        return 1
    fi
}

# Function to check TCP port
check_tcp() {
    local name=$1
    local host=$2
    local port=$3

    echo -n "  • $name: "
    if timeout 5 bash -c "</dev/tcp/$host/$port" >/dev/null 2>&1; then
        echo "✅ OK"
    else
        echo "❌ FAILED"
        return 1
    fi
}

# Check infrastructure services
echo "🗄️ Infrastructure Services:"
check_tcp "PostgreSQL" localhost 5432
check_tcp "Redis" localhost 6379
check_http "Traefik Dashboard" "http://localhost:8080/api/rawdata" 200

# Check application services
echo ""
echo "⚙️ Application Services:"
check_http "API Gateway" "https://api.dev.midominio.com/health" 200
check_http "Frontend" "https://dev.midominio.com" 200
check_http "Admin" "https://admin.dev.midominio.com" 200

# Check monitoring services
echo ""
echo "📊 Monitoring Services:"
check_http "Prometheus" "https://prometheus.dev.midominio.com/-/healthy" 200
check_http "Grafana" "https://grafana.dev.midominio.com/api/health" 200
check_http "Jaeger" "https://jaeger.dev.midominio.com" 200

echo ""
echo "✅ All services are healthy!"
```

### Database Reset Script

```bash
#!/bin/bash
# scripts/reset-db.sh

set -e

echo "🗄️ Resetting database..."

# Confirmación
read -p "⚠️  This will destroy all data. Are you sure? (y/N): " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Aborted"
    exit 1
fi

# Parar servicios que usan la base de datos
echo "🛑 Stopping application services..."
docker-compose stop api-gateway auth-service products-service orders-service payments-service notifications-service

# Parar y eliminar base de datos
echo "🗑️ Removing database container and volume..."
docker-compose stop postgres
docker-compose rm -f postgres
docker volume rm ecommerce_postgres-data

# Recrear base de datos
echo "🏗️ Creating fresh database..."
docker-compose up -d postgres

# Esperar a que esté lista
echo "⏳ Waiting for database to be ready..."
while ! docker-compose exec postgres pg_isready -U ecommerce_user -d ecommerce >/dev/null 2>&1; do
    sleep 1
done

# Ejecutar migraciones
echo "🔄 Running migrations..."
# docker-compose exec api-gateway npm run migration:run

# Seed data (opcional)
echo "🌱 Seeding initial data..."
# docker-compose exec api-gateway npm run seed

# Reiniciar servicios
echo "🚀 Starting application services..."
docker-compose up -d

echo "✅ Database reset complete!"
```

## 🔧 Comandos de Desarrollo

### Makefile para Desarrollo

```makefile
# Makefile
.PHONY: help dev-start dev-stop dev-restart dev-logs dev-shell health

# Default target
help:
	@echo "🛠️  Available commands:"
	@echo "  dev-start    - Start development environment"
	@echo "  dev-stop     - Stop all services"
	@echo "  dev-restart  - Restart all services"
	@echo "  dev-logs     - Show logs for all services"
	@echo "  dev-shell    - Open shell in specified service"
	@echo "  health       - Check service health"
	@echo "  reset-db     - Reset database (destructive!)"
	@echo "  clean        - Clean up containers and volumes"

dev-start:
	@echo "🚀 Starting development environment..."
	docker-compose up -d
	@echo "✅ Development environment started!"
	@echo "📱 Frontend: https://dev.midominio.com"
	@echo "🔧 Admin: https://admin.dev.midominio.com"
	@echo "🌐 API: https://api.dev.midominio.com"

dev-stop:
	@echo "🛑 Stopping development environment..."
	docker-compose down

dev-restart:
	@echo "🔄 Restarting development environment..."
	docker-compose restart

dev-logs:
	docker-compose logs -f

dev-shell:
	@read -p "Enter service name: " service; \
	docker-compose exec $$service sh

health:
	@./scripts/health-check.sh

reset-db:
	@./scripts/reset-db.sh

clean:
	@echo "🧹 Cleaning up containers and volumes..."
	docker-compose down -v --remove-orphans
	docker system prune -f
```

## 🎯 Checklist de Implementación

### ✅ **Setup Inicial**

- [ ] Copiar configuraciones Docker a tu proyecto
- [ ] Configurar variables en .env
- [ ] Actualizar dominios y subdominios
- [ ] Ejecutar script de setup
- [ ] Verificar servicios con health check

### ✅ **Desarrollo Local**

- [ ] Hot-reload funcionando en frontend
- [ ] Hot-reload funcionando en backend
- [ ] Base de datos inicializada
- [ ] Redis conectado y funcionando
- [ ] SSL automático con Traefik

### ✅ **Observabilidad**

- [ ] Prometheus recolectando métricas
- [ ] Grafana con dashboards básicos
- [ ] Logs centralizados en Loki
- [ ] Jaeger para distributed tracing
- [ ] Alertas básicas configuradas

### ✅ **Productividad**

- [ ] Scripts de automatización
- [ ] Comandos make para tareas comunes
- [ ] Debugging configurado
- [ ] Database management con PgAdmin
- [ ] Documentation actualizada

---

**🎯 Próximo paso**: Con el entorno local funcionando, continúa con [Deployment Strategies](./deployment.md) para configurar el pipeline de deployment a producción.

**💡 Tip**: El entorno Docker local debe replicar lo más posible tu entorno de producción. Esto reduce significativamente los bugs de deployment.
