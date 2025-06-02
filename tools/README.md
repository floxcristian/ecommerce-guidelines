# 🔧 Development Tools - Ecommerce Guidelines

Scripts y herramientas para automatizar tareas comunes de desarrollo, testing y deployment.

## 🛠️ Herramientas Disponibles

### 🚀 [Setup Scripts](./setup/)

**Scripts de configuración automática**

- `setup.sh` - Setup completo del entorno
- `install-deps.sh` - Instalar dependencias
- `configure-env.sh` - Configurar variables de entorno
- **Tiempo**: 5-10 minutos

### 🔄 [Deployment Scripts](./deploy/)

**Scripts de deployment automatizado**

- `deploy.sh` - Deploy a cualquier entorno
- `rollback.sh` - Rollback automático
- `health-check.sh` - Verificación post-deploy
- **Tiempo**: 2-5 minutos

### 🧪 [Testing Tools](./testing/)

**Herramientas de testing avanzado**

- `test-all.sh` - Ejecutar toda la suite de tests
- `load-test.sh` - Tests de carga con k6
- `security-scan.sh` - Scan de seguridad
- **Tiempo**: 10-30 minutos

### 📊 [Monitoring Tools](./monitoring/)

**Herramientas de monitoreo y alertas**

- `setup-monitoring.sh` - Configurar Prometheus + Grafana
- `alerts-config.sh` - Configurar alertas
- `dashboard-import.sh` - Importar dashboards
- **Tiempo**: 15-20 minutos

### 🔧 [Development Utilities](./dev/)

**Utilidades para desarrollo diario**

- `gen-service.sh` - Generar nuevo microservicio
- `db-migrate.sh` - Ejecutar migraciones
- `logs.sh` - Ver logs centralizados
- **Tiempo**: 1-3 minutos

## ⚡ Quick Start Tools

### Setup Completo (5 minutos)

```bash
# Setup automático completo
./tools/setup/setup.sh --env=development

# Verificar instalación
./tools/setup/verify.sh
```

### Deploy Rápido (2 minutos)

```bash
# Deploy a development
./tools/deploy/deploy.sh --env=dev

# Deploy a production
./tools/deploy/deploy.sh --env=prod --confirm
```

### Testing Completo (15 minutos)

```bash
# Ejecutar todos los tests
./tools/testing/test-all.sh

# Solo tests críticos
./tools/testing/test-critical.sh
```

## 📋 Índice de Scripts

### 🚀 Setup Scripts

| Script             | Descripción                      | Tiempo   | Dependencias    |
| ------------------ | -------------------------------- | -------- | --------------- |
| `setup.sh`         | Setup completo del proyecto      | 5-10 min | Docker, Node.js |
| `install-deps.sh`  | Instalar todas las dependencias  | 3-5 min  | npm/yarn        |
| `configure-env.sh` | Configurar variables de entorno  | 1-2 min  | -               |
| `setup-db.sh`      | Configurar base de datos inicial | 2-3 min  | PostgreSQL      |
| `setup-ssl.sh`     | Configurar certificados SSL      | 2-3 min  | OpenSSL         |

### 🔄 Deployment Scripts

| Script            | Descripción                     | Tiempo   | Dependencias  |
| ----------------- | ------------------------------- | -------- | ------------- |
| `deploy.sh`       | Deploy automático multi-entorno | 2-5 min  | kubectl, helm |
| `rollback.sh`     | Rollback a versión anterior     | 1-2 min  | kubectl       |
| `health-check.sh` | Verificar health de servicios   | 30 sec   | curl          |
| `backup.sh`       | Backup automático de datos      | 2-10 min | pg_dump       |
| `restore.sh`      | Restore de backup               | 1-5 min  | psql          |

### 🧪 Testing Scripts

| Script             | Descripción                | Tiempo    | Dependencias |
| ------------------ | -------------------------- | --------- | ------------ |
| `test-all.sh`      | Suite completa de tests    | 10-30 min | Jest, k6     |
| `test-unit.sh`     | Tests unitarios únicamente | 2-5 min   | Jest         |
| `test-e2e.sh`      | Tests end-to-end           | 5-15 min  | Cypress      |
| `load-test.sh`     | Tests de carga             | 5-10 min  | k6           |
| `security-scan.sh` | Scan de vulnerabilidades   | 3-5 min   | OWASP ZAP    |

### 📊 Monitoring Scripts

| Script                 | Descripción                   | Tiempo    | Dependencias    |
| ---------------------- | ----------------------------- | --------- | --------------- |
| `setup-monitoring.sh`  | Configurar stack de monitoreo | 15-20 min | Docker, kubectl |
| `import-dashboards.sh` | Importar dashboards Grafana   | 2-3 min   | curl            |
| `setup-alerts.sh`      | Configurar alertas            | 5-10 min  | Prometheus      |
| `check-metrics.sh`     | Verificar métricas            | 1 min     | curl            |

### 🔧 Development Scripts

| Script             | Descripción                 | Tiempo      | Dependencias   |
| ------------------ | --------------------------- | ----------- | -------------- |
| `gen-service.sh`   | Generar nuevo microservicio | 1-2 min     | NX CLI         |
| `gen-component.sh` | Generar componente Angular  | 30 sec      | Angular CLI    |
| `db-migrate.sh`    | Ejecutar migraciones        | 1 min       | TypeORM        |
| `logs.sh`          | Ver logs centralizados      | Instantáneo | kubectl/docker |
| `clean.sh`         | Limpiar archivos temporales | 30 sec      | -              |

## 🎯 Casos de Uso Comunes

### Nuevo Developer Onboarding

```bash
# 1. Setup completo del entorno
./tools/setup/setup.sh --env=development

# 2. Verificar que todo funciona
./tools/testing/test-critical.sh

# 3. Ejecutar aplicación
./tools/dev/start.sh
```

### Deploy a Producción

```bash
# 1. Ejecutar tests completos
./tools/testing/test-all.sh

# 2. Backup de producción
./tools/deploy/backup.sh --env=prod

# 3. Deploy con verificación
./tools/deploy/deploy.sh --env=prod --verify

# 4. Health check post-deploy
./tools/deploy/health-check.sh --env=prod
```

### Debugging Problemas

```bash
# 1. Ver logs de todos los servicios
./tools/dev/logs.sh --follow

# 2. Verificar métricas
./tools/monitoring/check-metrics.sh

# 3. Ejecutar health checks
./tools/deploy/health-check.sh --verbose
```

### Desarrollo de Nueva Feature

```bash
# 1. Crear nuevo servicio
./tools/dev/gen-service.sh --name=my-new-service

# 2. Generar componentes frontend
./tools/dev/gen-component.sh --name=my-component

# 3. Ejecutar tests durante desarrollo
./tools/testing/test-unit.sh --watch
```

## ⚙️ Configuración de Tools

### Variables de Entorno

```bash
# Copiar configuración de herramientas
cp tools/.env.example tools/.env

# Editar configuración
nano tools/.env
```

### Configuración Principal (tools/.env)

```bash
# Entornos
DEFAULT_ENV=development
PROD_CLUSTER=ecommerce-prod
STAGING_CLUSTER=ecommerce-staging

# Registries
DOCKER_REGISTRY=gcr.io/my-project
HELM_REGISTRY=oci://my-registry.com/helm

# Notificaciones
SLACK_WEBHOOK_URL=https://hooks.slack.com/...
EMAIL_ALERTS=alerts@mycompany.com

# Monitoreo
PROMETHEUS_URL=http://prometheus:9090
GRAFANA_URL=http://grafana:3000
```

## 🔒 Seguridad y Mejores Prácticas

### Permisos de Scripts

```bash
# Hacer scripts ejecutables
chmod +x tools/**/*.sh

# Solo owner puede modificar scripts críticos
chmod 750 tools/deploy/*.sh
chmod 750 tools/setup/*.sh
```

### Validaciones de Seguridad

- ✅ Todos los scripts validan parámetros
- ✅ Confirmación requerida para operaciones destructivas
- ✅ Logs de auditoría para todas las operaciones
- ✅ Timeouts para evitar scripts colgados
- ✅ Rollback automático en caso de fallo

### Uso en CI/CD

```yaml
# .github/workflows/ci.yml
- name: Run setup tools
  run: ./tools/setup/setup.sh --env=ci

- name: Run tests
  run: ./tools/testing/test-all.sh --ci-mode

- name: Deploy
  run: ./tools/deploy/deploy.sh --env=staging --auto-confirm
```

## 🚀 Roadmap de Tools

### ✅ Disponible Ahora

- [x] Setup automático completo
- [x] Deploy multi-entorno
- [x] Testing suite completa
- [x] Monitoreo básico

### 🔄 En Desarrollo

- [ ] AI-powered debugging tools
- [ ] Performance optimization scripts
- [ ] Automated security hardening
- [ ] Cost optimization tools

### 📅 Próximamente

- [ ] Multi-cloud deployment tools
- [ ] Advanced analytics scripts
- [ ] Compliance automation
- [ ] Disaster recovery tools

## 🔗 Enlaces de Navegación

### Documentación

- [📚 Docs](../docs/) - Guías técnicas
- [📖 Examples](../examples/) - Ejemplos funcionales
- [📦 Templates](../templates/) - Templates reutilizables

### Quick Links

- [🚀 Getting Started](../GETTING_STARTED.md) - Inicio rápido
- [📁 Project Structure](../PROJECT_STRUCTURE.md) - Estructura del proyecto

---

**🎯 Próximo paso**: Explora las herramientas específicas que necesites para automatizar tu workflow de desarrollo.
