# 📁 Estructura del Proyecto - Ecommerce Guidelines

Este archivo documenta la organización completa del repositorio.

## 📂 Estructura del Repositorio

```
ecommerce-guidelines/
├── README.md                           # Guía principal con navegación
├── GETTING_STARTED.md                  # Punto de entrada por rol
├── PROJECT_STRUCTURE.md               # Este archivo (documentación)
│
├── docs/                              # Guías principales organizadas
│   ├── frontend/                      # 🎨 Todo lo relacionado con frontend
│   │   ├── README.md                  # 📄 Overview frontend
│   │   ├── angular-optimization.md   # 📄 Performance y Core Web Vitals ✅
│   │   ├── state-management.md       # 📄 NgRx en profundidad
│   │   ├── pwa-seo.md                # 📄 PWA, SEO y meta tags
│   │   └── ui-design-system.md       # 📄 Componentes y design tokens
│   │
│   ├── backend/                      # ⚙️ Todo lo relacionado con backend
│   │   ├── README.md                 # 📄 Overview backend
│   │   ├── microservices-architecture.md  # 📄 Implementación completa ✅
│   │   ├── nestjs-patterns.md        # 📄 Patrones específicos de NestJS
│   │   ├── api-gateway.md            # 📄 Gateway en profundidad
│   │   ├── authentication.md         # 📄 Auth, JWT, RBAC
│   │   ├── event-driven.md           # 📄 BullMQ, eventos, workers
│   │   ├── database-design.md        # 📄 Base de datos y migraciones
│   │   └── resilience-patterns.md    # 📄 Circuit breaker, retry, etc
│   │
│   ├── infrastructure/               # 🏗️ Todo DevOps e infraestructura
│   │   ├── README.md                 # 📄 Overview infrastructure
│   │   ├── docker-development.md     # 📄 Entorno local optimizado ✅
│   │   ├── kubernetes-production.md  # 📄 K8s y GKE en profundidad
│   │   ├── ci-cd-pipelines.md       # 📄 GitHub Actions detallado
│   │   ├── monitoring-observability.md # 📄 Prometheus, Grafana, logs
│   │   ├── security-hardening.md     # 📄 Seguridad y best practices
│   │   └── terraform-iac.md         # 📄 Infrastructure as Code
│   │
│   └── architecture/                # 🏛️ Decisiones arquitectónicas
│       ├── README.md                # 📄 Overview de decisiones
│       ├── tech-stack-decisions.md  # 📄 Por qué elegimos cada tecnología
│       ├── scalability-patterns.md  # 📄 Patrones de escalabilidad
│       └── migration-strategies.md  # 📄 Cómo migrar sistemas existentes
│
├── examples/                        # 📖 Código de ejemplo funcional
│   ├── basic-setup/                # 📖 Setup mínimo funcional
│   ├── microservices-demo/         # 📖 Demo completo de microservicios
│   └── production-configs/          # 📖 Configuraciones reales
│
├── templates/                      # 📦 Templates listos para usar
│   ├── README.md                   # 📄 Documentación de templates ✅
│   ├── service-template/           # 📦 Template de microservicio
│   ├── frontend-template/          # 📦 Template de app Angular
│   └── infrastructure-template/    # 📦 Template de infraestructura
│
└── tools/                          # 🔧 Scripts y herramientas
    ├── setup.sh                   # 🔧 Script de setup automático
    ├── deploy.sh                  # 🔧 Script de deployment
    ├── migrate.sh                 # 🔧 Script de migración
    └── README.md                  # 📄 Documentación de tools
```

## 🔄 Navegación por Áreas

### Frontend Development

- **Angular Optimization**: Performance, PWA, SEO
- **State Management**: NgRx patterns avanzados
- **UI Design System**: Componentes reutilizables

### Backend Development

- **Microservices**: Arquitectura escalable con NestJS
- **API Gateway**: Routing, auth, rate limiting
- **Event-Driven**: BullMQ y comunicación asíncrona

### Infrastructure & DevOps

- **Docker Development**: Entorno local optimizado
- **Kubernetes Production**: Deployment escalable
- **CI/CD Pipelines**: Automatización completa

### Architecture Decisions

- **Tech Stack**: Justificación de tecnologías
- **Scalability Patterns**: Estrategias de crecimiento
- **Migration Strategies**: Transición de sistemas existentes

## 🛠️ Desarrollo y Implementación

### Código Funcional

- **examples/**: Demos completos listos para usar
- **templates/**: Boilerplates personalizables
- **tools/**: Scripts de automatización

### Flujo de Trabajo

1. Explorar área de interés en `docs/`
2. Revisar ejemplos en `examples/`
3. Usar templates de `templates/`
4. Automatizar con `tools/`
