# 🏗️ Infraestructura y Deployment - Guía Completa

Guía detallada para configurar la infraestructura completa del ecommerce, desde contenedores Docker hasta Kubernetes en producción, incluyendo CI/CD, monitoreo y seguridad.

## 🎯 Objetivos

- Infraestructura como código (IaC)
- Deployment automatizado
- Alta disponibilidad y escalabilidad
- Observabilidad completa
- Seguridad en múltiples capas

## 🏢 Arquitectura de Infraestructura

```mermaid
graph TB
    subgraph "Cloudflare"
        CF[WAF + CDN + DDoS]
    end

    subgraph "Desarrollo - VPS Vultr"
        VPS[Docker Compose]
        TRAEFIK_DEV[Traefik]
        REDIS_DEV[Redis]
        APPS_DEV[Apps + DBs]
    end

    subgraph "Producción - Google Cloud"
        LB[Load Balancer]
        GKE[Google Kubernetes Engine]

        subgraph "Namespaces"
            NS_PROD[ecommerce]
            NS_MONITORING[monitoring]
            NS_LOGGING[logging]
            NS_TRAEFIK[traefik]
        end

        subgraph "Databases"
            CLOUD_SQL[Cloud SQL PostgreSQL]
            REDIS_CLOUD[Memorystore Redis]
        end

        subgraph "Observabilidad"
            PROMETHEUS[Prometheus]
            GRAFANA[Grafana]
            LOKI[Loki]
            JAEGER[Jaeger]
        end
    end

    CF --> TRAEFIK_DEV
    CF --> LB
    LB --> GKE
    TRAEFIK_DEV --> APPS_DEV
    GKE --> CLOUD_SQL
    GKE --> REDIS_CLOUD
```

## 🛠️ Entorno de Desarrollo

### Docker Compose Completo

```yaml
# docker-compose.yml
version: "3.8"

networks:
  ecommerce:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
  letsencrypt:

services:
  # Reverse Proxy
  traefik:
    image: traefik:v3.0
    container_name: traefik
    restart: unless-stopped
    command:
      - "--api.dashboard=true"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=dev@floxcristian.cl"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--log.level=INFO"
      - "--accesslog=true"
      - "--metrics.prometheus=true"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080" # Dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.dev.floxcristian.cl`)"

  # Databases
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ecommerce
      POSTGRES_USER: ecommerce
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-dbs.sql:/docker-entrypoint-initdb.d/init-dbs.sql
    networks:
      - ecommerce
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ecommerce"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - ecommerce
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Frontend
  frontend:
    build:
      context: .
      dockerfile: apps/frontend/Dockerfile
      target: development
    container_name: frontend
    restart: unless-stopped
    volumes:
      - ./apps/frontend:/app
      - /app/node_modules
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`dev.floxcristian.cl`)"
      - "traefik.http.routers.frontend.entrypoints=websecure"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
      - "traefik.http.services.frontend.loadbalancer.server.port=4200"

  # Admin Panel
  admin:
    build:
      context: .
      dockerfile: apps/admin/Dockerfile
      target: development
    container_name: admin
    restart: unless-stopped
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.admin.rule=Host(`admin.dev.floxcristian.cl`)"
      - "traefik.http.routers.admin.entrypoints=websecure"
      - "traefik.http.routers.admin.tls.certresolver=letsencrypt"

  # API Gateway
  api-gateway:
    build:
      context: .
      dockerfile: apps/api-gateway/Dockerfile
      target: development
    container_name: api-gateway
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - JWT_SECRET=${JWT_SECRET}
      - DATABASE_URL=postgresql://ecommerce:${POSTGRES_PASSWORD}@postgres:5432/ecommerce
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.dev.floxcristian.cl`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"

  # Microservicios
  auth-service:
    build:
      context: .
      dockerfile: apps/auth-service/Dockerfile
      target: development
    container_name: auth-service
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://ecommerce:${POSTGRES_PASSWORD}@postgres:5432/auth_db
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce

  products-service:
    build:
      context: .
      dockerfile: apps/products-service/Dockerfile
      target: development
    container_name: products-service
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://ecommerce:${POSTGRES_PASSWORD}@postgres:5432/products_db
    depends_on:
      - postgres
    networks:
      - ecommerce

  orders-service:
    build:
      context: .
      dockerfile: apps/orders-service/Dockerfile
      target: development
    container_name: orders-service
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://ecommerce:${POSTGRES_PASSWORD}@postgres:5432/orders_db
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - ecommerce

  payments-service:
    build:
      context: .
      dockerfile: apps/payments-service/Dockerfile
      target: development
    container_name: payments-service
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://ecommerce:${POSTGRES_PASSWORD}@postgres:5432/payments_db
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
    depends_on:
      - postgres
    networks:
      - ecommerce

  # Workers
  workers:
    build:
      context: .
      dockerfile: apps/workers/Dockerfile
      target: development
    container_name: workers
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379
      - EMAIL_SERVICE_URL=${EMAIL_SERVICE_URL}
    depends_on:
      - redis
    networks:
      - ecommerce

  # Monitoring (Desarrollo)
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.prometheus.rule=Host(`prometheus.dev.floxcristian.cl`)"
      - "traefik.http.routers.prometheus.entrypoints=websecure"
      - "traefik.http.routers.prometheus.tls.certresolver=letsencrypt"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - ./monitoring/grafana:/var/lib/grafana
    networks:
      - ecommerce
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.grafana.rule=Host(`grafana.dev.floxcristian.cl`)"
      - "traefik.http.routers.grafana.entrypoints=websecure"
      - "traefik.http.routers.grafana.tls.certresolver=letsencrypt"
```

### Archivo de Variables de Entorno

```bash
# .env
POSTGRES_PASSWORD=your_postgres_password
REDIS_PASSWORD=your_redis_password
JWT_SECRET=your_jwt_secret_key
STRIPE_SECRET_KEY=sk_test_your_stripe_key
GRAFANA_PASSWORD=admin_password
EMAIL_SERVICE_URL=your_email_service_url
```

### Script de Inicialización de DBs

```sql
-- scripts/init-dbs.sql
CREATE DATABASE auth_db;
CREATE DATABASE products_db;
CREATE DATABASE orders_db;
CREATE DATABASE payments_db;

-- Crear usuario específico para cada servicio (opcional)
CREATE USER auth_user WITH PASSWORD 'auth_password';
CREATE USER products_user WITH PASSWORD 'products_password';
CREATE USER orders_user WITH PASSWORD 'orders_password';
CREATE USER payments_user WITH PASSWORD 'payments_password';

-- Asignar permisos
GRANT ALL PRIVILEGES ON DATABASE auth_db TO auth_user;
GRANT ALL PRIVILEGES ON DATABASE products_db TO products_user;
GRANT ALL PRIVILEGES ON DATABASE orders_db TO orders_user;
GRANT ALL PRIVILEGES ON DATABASE payments_db TO payments_user;
```

## ☁️ Entorno de Producción - Google Cloud

### Terraform para Infraestructura

```hcl
# terraform/main.tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

# Variables
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "GCP Region"
  type        = string
  default     = "us-central1"
}

variable "cluster_name" {
  description = "GKE Cluster Name"
  type        = string
  default     = "ecommerce-cluster"
}

# VPC Network
resource "google_compute_network" "vpc" {
  name                    = "ecommerce-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "ecommerce-subnet"
  ip_cidr_range = "10.0.0.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id
}

# GKE Cluster
resource "google_container_cluster" "primary" {
  name     = var.cluster_name
  location = var.region

  # We can't create a cluster with no node pool defined, but we want to only use
  # separately managed node pools. So we create the smallest possible default
  # node pool and immediately delete it.
  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.vpc.name
  subnetwork = google_compute_subnetwork.subnet.name

  # Enable Workload Identity
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  # Enable network policy
  network_policy {
    enabled = true
  }

  # Enable maintenance window
  maintenance_policy {
    daily_maintenance_window {
      start_time = "03:00"
    }
  }
}

# Node Pool
resource "google_container_node_pool" "primary_nodes" {
  name       = "main-pool"
  location   = var.region
  cluster    = google_container_cluster.primary.name
  node_count = 2

  node_config {
    preemptible  = false
    machine_type = "e2-standard-2"

    service_account = google_service_account.gke_sa.email
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    labels = {
      env = "production"
    }

    tags = ["gke-node", "ecommerce-gke-node"]
  }

  autoscaling {
    min_node_count = 1
    max_node_count = 5
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }
}

# Service Account for GKE
resource "google_service_account" "gke_sa" {
  account_id   = "gke-service-account"
  display_name = "GKE Service Account"
}

# Cloud SQL PostgreSQL
resource "google_sql_database_instance" "postgres" {
  name             = "ecommerce-postgres"
  database_version = "POSTGRES_15"
  region          = var.region

  settings {
    tier = "db-f1-micro"

    backup_configuration {
      enabled                        = true
      start_time                     = "02:00"
      location                       = var.region
      point_in_time_recovery_enabled = true
    }

    database_flags {
      name  = "log_statement"
      value = "all"
    }

    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id
    }
  }

  deletion_protection = true
}

# Redis (Memorystore)
resource "google_redis_instance" "redis" {
  name           = "ecommerce-redis"
  tier           = "STANDARD_HA"
  memory_size_gb = 1
  region         = var.region

  authorized_network = google_compute_network.vpc.id
  connect_mode       = "PRIVATE_SERVICE_ACCESS"

  redis_configs = {
    maxmemory-policy = "allkeys-lru"
  }
}

# Outputs
output "cluster_endpoint" {
  value = google_container_cluster.primary.endpoint
}

output "cluster_ca_certificate" {
  value = google_container_cluster.primary.master_auth[0].cluster_ca_certificate
}

output "postgres_connection_name" {
  value = google_sql_database_instance.postgres.connection_name
}

output "redis_host" {
  value = google_redis_instance.redis.host
}
```

### Kubernetes Manifests

#### Namespace y RBAC

```yaml
# k8s/base/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
  labels:
    name: ecommerce
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    name: monitoring
---
apiVersion: v1
kind: Namespace
metadata:
  name: traefik
  labels:
    name: traefik
```

#### ConfigMaps y Secrets

```yaml
# k8s/base/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ecommerce
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  API_GATEWAY_URL: "https://api.floxcristian.cl"
  FRONTEND_URL: "https://floxcristian.cl"
---
# k8s/base/sealed-secret.yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: app-secrets
  namespace: ecommerce
spec:
  encryptedData:
    JWT_SECRET: <encrypted-value>
    POSTGRES_PASSWORD: <encrypted-value>
    REDIS_PASSWORD: <encrypted-value>
    STRIPE_SECRET_KEY: <encrypted-value>
```

#### Deployments

```yaml
# k8s/apps/api-gateway.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: ecommerce
  labels:
    app: api-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      containers:
        - name: api-gateway
          image: gcr.io/PROJECT_ID/api-gateway:TAG
          ports:
            - containerPort: 3000
              name: http
          env:
            - name: NODE_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: NODE_ENV
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: JWT_SECRET
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: ecommerce
  labels:
    app: api-gateway
spec:
  selector:
    app: api-gateway
  ports:
    - port: 80
      targetPort: 3000
      name: http
```

### Helm Charts

#### Chart.yaml

```yaml
# helm/charts/api-gateway/Chart.yaml
apiVersion: v2
name: api-gateway
description: API Gateway for ecommerce platform
type: application
version: 0.1.0
appVersion: "1.0.0"
```

#### Values.yaml

```yaml
# helm/charts/api-gateway/values.yaml
replicaCount: 3

image:
  repository: gcr.io/PROJECT_ID/api-gateway
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: "traefik"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    traefik.ingress.kubernetes.io/router.middlewares: "traefik-auth@kubernetescrd"
  hosts:
    - host: api.floxcristian.cl
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.floxcristian.cl

resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

nodeSelector: {}
tolerations: []
affinity: {}

env:
  NODE_ENV: production
  LOG_LEVEL: info

secrets:
  JWT_SECRET: ""
  DATABASE_URL: ""
  REDIS_URL: ""
```

## 🚀 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  GKE_CLUSTER: ecommerce-cluster
  GKE_ZONE: us-central1
  IMAGE_TAG: ${{ github.sha }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: |
          npx nx test api-gateway
          npx nx test auth-service
          npx nx test products-service
          npx nx test orders-service
          npx nx test payments-service

      - name: Run e2e tests
        run: npx nx e2e frontend-e2e

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    strategy:
      matrix:
        service:
          [
            frontend,
            admin,
            api-gateway,
            auth-service,
            products-service,
            orders-service,
            payments-service,
            workers,
          ]

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npx nx build ${{ matrix.service }} --prod

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
        with:
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Configure Docker
        run: gcloud auth configure-docker

      - name: Build Docker image
        run: |
          docker build \
            -t gcr.io/$PROJECT_ID/${{ matrix.service }}:$IMAGE_TAG \
            -t gcr.io/$PROJECT_ID/${{ matrix.service }}:latest \
            -f apps/${{ matrix.service }}/Dockerfile .

      - name: Push Docker image
        run: |
          docker push gcr.io/$PROJECT_ID/${{ matrix.service }}:$IMAGE_TAG
          docker push gcr.io/$PROJECT_ID/${{ matrix.service }}:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
        with:
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          project_id: ${{ secrets.GCP_PROJECT_ID }}

      - name: Get GKE credentials
        run: |
          gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE

      - name: Install Helm
        uses: azure/setup-helm@v3
        with:
          version: "3.10.0"

      - name: Install Helmfile
        run: |
          wget https://github.com/roboll/helmfile/releases/download/v0.157.0/helmfile_linux_amd64
          chmod +x helmfile_linux_amd64
          sudo mv helmfile_linux_amd64 /usr/local/bin/helmfile

      - name: Deploy with Helmfile
        run: |
          export IMAGE_TAG=${{ github.sha }}
          helmfile -e production apply

      - name: Verify deployment
        run: |
          kubectl rollout status deployment/api-gateway -n ecommerce
          kubectl rollout status deployment/frontend -n ecommerce
          kubectl get pods -n ecommerce
```

## 📊 Monitoreo y Observabilidad

### Prometheus Configuration

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "traefik"
    static_configs:
      - targets: ["traefik:8080"]

  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

  - job_name: "node-exporter"
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_endpoints_name]
        action: keep
        regex: node-exporter
```

### Alerting Rules

```yaml
# monitoring/rules/alerts.yml
groups:
  - name: ecommerce.rules
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: High error rate detected
          description: "Error rate is {{ $value }} errors per second"

      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: Pod is crash looping
          description: "Pod {{ $labels.pod }} is restarting frequently"

      - alert: HighMemoryUsage
        expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: Container memory usage is high
          description: "Container {{ $labels.container }} memory usage is above 90%"
```

## 🔗 Próximos Pasos

### Enlaces Relacionados

- [🚀 Guía Completa Full-Stack](./new.md)
- [⚙️ Microservicios con NestJS](./backend-microservices.md)
- [🎨 Frontend Angular](./frontend-optimization.md) (próximamente)

### Próximos Temas

- [ ] Configuración avanzada de Istio Service Mesh
- [ ] Estrategias de Blue/Green y Canary deployments
- [ ] Backup y disaster recovery
- [ ] Optimización de costos en GCP
- [ ] Multi-region deployment

---

> 💡 **Tip**: Comienza con el entorno de desarrollo local usando Docker Compose. Una vez estable, migra gradualmente a Kubernetes en producción.
