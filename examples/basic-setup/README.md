# ⚡ Basic Setup - Ecommerce Mínimo Funcional

Setup mínimo para tener un ecommerce completo funcionando en 15 minutos con Docker Compose.

## 🎯 Qué incluye este ejemplo

### 🏗️ Servicios Core

- **API Gateway** (Puerto 3000) - Routing y autenticación
- **Frontend Angular** (Puerto 4200) - App cliente optimizada
- **PostgreSQL** (Puerto 5432) - Base de datos principal
- **Redis** (Puerto 6379) - Cache y sesiones

### 🔧 Características

- ✅ Autenticación JWT funcional
- ✅ CRUD de productos completo
- ✅ Carrito de compras persistente
- ✅ UI Angular con Material Design
- ✅ SSL automático con Traefik
- ✅ Hot reload para desarrollo

## 🚀 Quick Start (15 minutos)

### Paso 1: Preparar el entorno

```bash
# Clonar o copiar los archivos
cd examples/basic-setup/

# Copiar variables de entorno
cp .env.example .env

# Editar variables si es necesario
nano .env
```

### Paso 2: Levantar servicios

```bash
# Construir y levantar todos los servicios
docker-compose up -d

# Ver logs de todos los servicios
docker-compose logs -f

# Ver estado de los servicios
docker-compose ps
```

### Paso 3: Verificar funcionamiento

```bash
# Verificar API Gateway
curl http://localhost:3000/health

# Verificar frontend
curl http://localhost:4200

# Verificar base de datos
docker-compose exec postgres psql -U ecommerce -d ecommerce -c "\dt"
```

### Paso 4: Probar la aplicación

```bash
# Abrir frontend en el navegador
open http://localhost:4200

# Probar login (usuario demo)
# Email: admin@example.com
# Password: admin123

# Probar API directamente
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "admin123"}'
```

## 📁 Estructura de Archivos

```
basic-setup/
├── docker-compose.yml         # Configuración principal
├── .env.example              # Variables de entorno
├── .env                      # Variables locales (crear)
├── README.md                 # Esta documentación
│
├── configs/                  # Configuraciones
│   ├── nginx.conf           # Configuración Nginx
│   ├── traefik.yml          # Configuración Traefik
│   └── init-db.sql          # Script inicial de BD
│
├── apps/                    # Código de aplicaciones
│   ├── api-gateway/         # Código del gateway
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── src/
│   │
│   └── frontend/            # Código del frontend
│       ├── Dockerfile
│       ├── package.json
│       └── src/
│
└── scripts/                 # Scripts útiles
    ├── setup.sh            # Script de setup automático
    ├── reset.sh            # Reset completo
    └── backup.sh           # Backup de datos
```

## ⚙️ Configuración Detallada

### Variables de Entorno (.env)

```bash
# Base de datos
POSTGRES_DB=ecommerce
POSTGRES_USER=ecommerce
POSTGRES_PASSWORD=secure_password_123

# Redis
REDIS_PASSWORD=redis_password_123

# JWT
JWT_SECRET=your_super_secret_jwt_key_here

# Aplicación
NODE_ENV=development
API_PORT=3000
FRONTEND_PORT=4200

# URLs
API_URL=http://localhost:3000
FRONTEND_URL=http://localhost:4200
```

### Puertos Utilizados

| Servicio              | Puerto | URL Local             |
| --------------------- | ------ | --------------------- |
| **Frontend**          | 4200   | http://localhost:4200 |
| **API Gateway**       | 3000   | http://localhost:3000 |
| **PostgreSQL**        | 5432   | localhost:5432        |
| **Redis**             | 6379   | localhost:6379        |
| **Traefik Dashboard** | 8080   | http://localhost:8080 |

## 🔍 Testing y Verificación

### Health Checks

```bash
# API Gateway health
curl http://localhost:3000/health

# Base de datos
docker-compose exec postgres pg_isready -U ecommerce

# Redis
docker-compose exec redis redis-cli ping
```

### Datos de Prueba

```bash
# El setup incluye datos de ejemplo:
# - Usuario admin: admin@example.com / admin123
# - 10 productos de muestra
# - 3 categorías básicas

# Verificar datos
curl http://localhost:3000/products
curl http://localhost:3000/categories
```

### APIs Disponibles

```bash
# Autenticación
POST /auth/login
POST /auth/register
POST /auth/logout

# Productos
GET /products
GET /products/:id
POST /products (requiere auth admin)
PUT /products/:id (requiere auth admin)
DELETE /products/:id (requiere auth admin)

# Carrito
GET /cart (requiere auth)
POST /cart/items (requiere auth)
PUT /cart/items/:id (requiere auth)
DELETE /cart/items/:id (requiere auth)

# Usuarios
GET /users/profile (requiere auth)
PUT /users/profile (requiere auth)
```

## 🛠️ Comandos Útiles

### Desarrollo

```bash
# Ver logs en tiempo real
docker-compose logs -f api-gateway
docker-compose logs -f frontend

# Reiniciar un servicio específico
docker-compose restart api-gateway

# Acceder a contenedor
docker-compose exec api-gateway sh
docker-compose exec postgres psql -U ecommerce
```

### Mantenimiento

```bash
# Backup de base de datos
docker-compose exec postgres pg_dump -U ecommerce ecommerce > backup.sql

# Restore de base de datos
docker-compose exec -T postgres psql -U ecommerce ecommerce < backup.sql

# Limpiar datos y empezar fresh
docker-compose down -v
docker-compose up -d
```

### Debugging

```bash
# Ver estado de todos los servicios
docker-compose ps

# Ver uso de recursos
docker stats

# Inspect de red
docker network ls
docker network inspect basic-setup_default
```

## 🚀 Próximos Pasos

### Para continuar aprendiendo:

1. **[Microservices Demo](../microservices-demo/)** - Ver implementación completa
2. **[Production Configs](../production-configs/)** - Preparar para producción

### Para personalizar:

1. **Modificar el frontend** - Edita `apps/frontend/src/`
2. **Añadir endpoints** - Modifica `apps/api-gateway/src/`
3. **Configurar dominio** - Actualiza `docker-compose.yml` con tu dominio

### Para producción:

1. **Cambiar passwords** - Usa passwords seguros en `.env`
2. **Configurar SSL** - Añade certificados reales
3. **Optimizar build** - Usar builds de producción

## ❓ Troubleshooting

### Problema: Servicios no levantan

```bash
# Verificar puertos libres
netstat -tulpn | grep -E "3000|4200|5432|6379"

# Limpiar y reintentar
docker-compose down
docker-compose up -d
```

### Problema: Base de datos no conecta

```bash
# Verificar logs de PostgreSQL
docker-compose logs postgres

# Verificar variables de entorno
docker-compose exec api-gateway env | grep POSTGRES
```

### Problema: Frontend no carga

```bash
# Verificar build del frontend
docker-compose logs frontend

# Verificar conectividad con API
curl http://localhost:3000/health
```

---

**🎯 ¡Éxito!** Ahora tienes un ecommerce completo funcionando. Prueba todas las funcionalidades y cuando estés listo, avanza al [Microservices Demo](../microservices-demo/).
