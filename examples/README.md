# 📖 Ejemplos Funcionales - Ecommerce Guidelines

Esta carpeta contiene ejemplos de código funcional que puedes usar directamente para implementar el stack completo de ecommerce.

## 🗂️ Estructura de Ejemplos

### ⚡ [Basic Setup](./basic-setup/)

**Setup mínimo funcional para empezar rápidamente**

- Docker Compose básico con todos los servicios
- Variables de entorno configuradas
- Scripts de inicialización
- **Tiempo de setup**: 15 minutos

### 🏗️ [Microservices Demo](./microservices-demo/)

**Implementación completa de microservicios**

- API Gateway + 5 microservicios funcionales
- Event-driven architecture con BullMQ
- Base de datos por servicio
- **Tiempo de setup**: 45 minutos

### 🚀 [Production Configs](./production-configs/)

**Configuraciones reales para producción**

- Helm charts para Kubernetes
- Terraform modules para GCP
- Configuraciones de monitoreo
- **Tiempo de setup**: 2-3 horas

## 🎯 Flujo de Aprendizaje Recomendado

### Para Principiantes

```
1. Basic Setup → Entender la arquitectura básica
2. Microservices Demo → Ver patrones en acción
3. Production Configs → Preparar para producción
```

### Para Experimentados

```
1. Microservices Demo → Analizar patrones avanzados
2. Production Configs → Implementar en cloud
3. Basic Setup → Referencia para nuevos proyectos
```

## 🛠️ Prerrequisitos

### Software Requerido

- **Docker** & **Docker Compose** (20.10+)
- **Node.js** (18+)
- **Git**

### Para Producción

- **kubectl** (1.25+)
- **Helm** (3.10+)
- **Terraform** (1.5+)
- Cuenta en **Google Cloud** o **AWS**

## ⚡ Quick Start

### 🚀 Setup Ultra Rápido (5 minutos)

```bash
# 1. Clonar y entrar al basic setup
cd examples/basic-setup/

# 2. Copiar variables de entorno
cp .env.example .env

# 3. Levantar servicios
docker-compose up -d

# 4. Verificar
curl http://localhost:3000/health
```

### 🏗️ Demo Completo (45 minutos)

```bash
# 1. Setup microservicios
cd examples/microservices-demo/

# 2. Instalar dependencias
npm install

# 3. Configurar y levantar
npm run setup
npm run dev

# 4. Probar APIs
npm run test:e2e
```

### ☁️ Producción (2-3 horas)

```bash
# 1. Configurar infraestructura
cd examples/production-configs/

# 2. Configurar Terraform
cd terraform/
terraform init
terraform plan

# 3. Crear infraestructura
terraform apply

# 4. Deploy con Helm
cd ../helm/
helmfile apply
```

## 📊 Comparación de Ejemplos

| Ejemplo                | Complejidad     | Servicios   | Bases de Datos               | Observabilidad | Tiempo Setup |
| ---------------------- | --------------- | ----------- | ---------------------------- | -------------- | ------------ |
| **Basic Setup**        | ⭐ Básico       | 3 core      | PostgreSQL + Redis           | Básico         | 15 min       |
| **Microservices Demo** | ⭐⭐⭐ Avanzado | 6 completos | PostgreSQL + Redis + MongoDB | Completo       | 45 min       |
| **Production Configs** | ⭐⭐⭐⭐ Expert | Escalable   | Cloud SQL + Memorystore      | Enterprise     | 2-3 hrs      |

## 🎯 Casos de Uso por Ejemplo

### 📖 Basic Setup - Ideal para:

- ✅ Aprender la arquitectura básica
- ✅ Desarrollo local rápido
- ✅ Prototipos y MVPs
- ✅ Demos y presentaciones

### 🏗️ Microservices Demo - Ideal para:

- ✅ Entender patrones avanzados
- ✅ Desarrollo de features complejas
- ✅ Testing de integración
- ✅ Capacitación de equipos

### 🚀 Production Configs - Ideal para:

- ✅ Deployments reales
- ✅ Escalabilidad enterprise
- ✅ Observabilidad completa
- ✅ Compliance y auditorías

## 🔗 Enlaces de Navegación

### Documentación Relacionada

- [📚 Docs Frontend](../docs/frontend/) - Optimización Angular
- [📚 Docs Backend](../docs/backend/) - Microservicios NestJS
- [📚 Docs Infrastructure](../docs/infrastructure/) - Docker + K8s

### Templates Listos

- [📦 Templates](../templates/) - Templates reutilizables para proyectos

### Herramientas

- [🔧 Tools](../tools/) - Scripts y automatización

---

**🎯 Próximo paso**: Comienza con [Basic Setup](./basic-setup/) para ver el stack en acción.
