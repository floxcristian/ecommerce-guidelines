# 📦 Templates Reutilizables - Ecommerce Guidelines

Templates listos para usar que puedes copiar directamente a tu proyecto para acelerar el desarrollo.

## 🗂️ Templates Disponibles

### ⚙️ [Service Template](./service-template/)

**Template completo de microservicio NestJS**

- Estructura base con mejores prácticas
- Dockerfile optimizado multi-stage
- Helm chart para Kubernetes
- Tests unitarios y de integración
- **Tiempo de setup**: 10 minutos

### 🎨 [Frontend Template](./frontend-template/)

**Template de aplicación Angular optimizada**

- NX workspace configurado
- PWA y SEO optimizado
- NgRx state management
- Design system incluido
- **Tiempo de setup**: 15 minutos

### 🏗️ [Infrastructure Template](./infrastructure-template/)

**Template completo de infraestructura**

- Terraform modules para GCP/AWS
- Kubernetes manifests
- GitHub Actions workflows
- Monitoring y observabilidad
- **Tiempo de setup**: 30 minutos

## 🎯 Cómo usar los Templates

### Método 1: Copiar y Personalizar

```bash
# 1. Copiar template deseado
cp -r templates/service-template/ my-new-service/

# 2. Personalizar configuración
cd my-new-service/
find . -name "*.json" -o -name "*.ts" -o -name "*.yml" | xargs sed -i 's/template-service/my-new-service/g'

# 3. Instalar dependencias
npm install

# 4. Ejecutar
npm run dev
```

### Método 2: Usar Generator (Recomendado)

```bash
# Instalar generador global
npm install -g @ecommerce-platform/generator

# Generar nuevo servicio
ecommerce generate service my-new-service

# Generar frontend
ecommerce generate frontend my-frontend-app

# Generar infraestructura
ecommerce generate infrastructure my-infrastructure
```

## 📊 Comparación de Templates

| Template           | Tecnologías                 | Características               | Complejidad     | Tiempo Setup |
| ------------------ | --------------------------- | ----------------------------- | --------------- | ------------ |
| **Service**        | NestJS + PostgreSQL + Redis | API REST + Events + Tests     | ⭐⭐ Medio      | 10 min       |
| **Frontend**       | Angular + NgRx + PWA        | App completa + SEO            | ⭐⭐⭐ Avanzado | 15 min       |
| **Infrastructure** | Terraform + K8s + CI/CD     | Cloud native + Observabilidad | ⭐⭐⭐⭐ Expert | 30 min       |

## 🎯 Casos de Uso por Template

### ⚙️ Service Template - Ideal para:

- ✅ Crear nuevos microservicios rápidamente
- ✅ Mantener consistencia en la arquitectura
- ✅ Onboarding de nuevos developers
- ✅ Prototipos de APIs

### 🎨 Frontend Template - Ideal para:

- ✅ Nuevas aplicaciones Angular
- ✅ Admin panels y dashboards
- ✅ PWAs de ecommerce
- ✅ Mobile apps con Ionic

### 🏗️ Infrastructure Template - Ideal para:

- ✅ Setup inicial de proyectos
- ✅ Múltiples entornos (dev/staging/prod)
- ✅ Compliance y governance
- ✅ Teams de DevOps

## 🛠️ Prerrequisitos

### Para Service Template

- **Node.js** (18+)
- **Docker** (20.10+)
- **PostgreSQL** (local o Docker)

### Para Frontend Template

- **Node.js** (18+)
- **Angular CLI** (16+)
- **NX CLI** (16+)

### Para Infrastructure Template

- **Terraform** (1.5+)
- **kubectl** (1.25+)
- **Helm** (3.10+)
- Cuenta en **GCP** o **AWS**

## 🔧 Personalización

### Variables de Template

Cada template incluye variables que debes personalizar:

```bash
# Variables comunes en todos los templates
{{PROJECT_NAME}}       # Nombre del proyecto
{{SERVICE_NAME}}       # Nombre del servicio
{{AUTHOR_NAME}}        # Tu nombre
{{AUTHOR_EMAIL}}       # Tu email
{{COMPANY_NAME}}       # Nombre de la empresa
{{PORT}}               # Puerto del servicio
{{DATABASE_NAME}}      # Nombre de la base de datos
```

### Script de Personalización

```bash
# Usar el script incluido para personalizar
./scripts/customize.sh \
  --project-name="Mi Ecommerce" \
  --service-name="products-service" \
  --author-name="Tu Nombre" \
  --author-email="tu@email.com"
```

## 📚 Estructura Estándar

Todos los templates siguen la misma estructura base:

```
template-name/
├── README.md                 # Documentación específica
├── .env.example             # Variables de entorno
├── package.json             # Dependencias Node.js
├── Dockerfile               # Container optimizado
├── docker-compose.yml       # Para desarrollo local
│
├── src/                     # Código fuente
│   ├── main.ts             # Punto de entrada
│   ├── app.module.ts       # Módulo principal
│   └── modules/            # Módulos específicos
│
├── test/                    # Tests
│   ├── unit/               # Tests unitarios
│   ├── integration/        # Tests de integración
│   └── e2e/                # Tests end-to-end
│
├── helm/                    # Charts de Kubernetes
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│
├── .github/                 # GitHub Actions
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
│
└── scripts/                 # Scripts útiles
    ├── setup.sh
    ├── test.sh
    └── deploy.sh
```

## 🚀 Roadmap de Templates

### ✅ Disponibles Ahora

- [x] Service Template (NestJS)
- [x] Frontend Template (Angular)
- [x] Infrastructure Template (K8s)

### 🔄 En Desarrollo

- [ ] Mobile Template (Ionic + Capacitor)
- [ ] Worker Template (BullMQ)
- [ ] Gateway Template (Kong + Plugins)
- [ ] Monorepo Template (NX completo)

### 📅 Próximamente

- [ ] Lambda Template (Serverless)
- [ ] Database Template (Migrations + Seeds)
- [ ] Analytics Template (Data pipeline)
- [ ] Security Template (Auth + RBAC)

## 🔗 Enlaces de Navegación

### Ejemplos Funcionales

- [📖 Examples](../examples/) - Código funcional para probar

### Documentación

- [📚 Docs](../docs/) - Guías técnicas detalladas
- [🚀 Getting Started](../GETTING_STARTED.md) - Inicio rápido

### Herramientas

- [🔧 Tools](../tools/) - Scripts y automatización

---

**🎯 Próximo paso**: Elige el template que necesites y personalízalo para tu proyecto específico.
