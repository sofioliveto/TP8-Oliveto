# TP8: Contenedores y Automatización - Setup Guide

## 🎯 Objetivo

Este documento describe cómo configurar el proyecto TP8 con contenedores Docker y CI/CD automático usando **servicios 100% gratuitos**.

## 🏗️ Arquitectura

```
┌──────────────────────────────────────────────────────────┐
│  GitHub Repository (sofioliveto/TP8-Oliveto)           │
│  ├─ backend/ (Node.js + Express + SQLite)               │
│  ├─ frontend/ (HTML/CSS/JS Vanilla)                     │
│  └─ .github/workflows/ci-cd-containers.yml              │
└──────────────────────────────────────────────────────────┘
                         ↓
         ┌───────────────────────────────┐
         │   GitHub Actions (CI/CD)      │
         │   ├─ Build & Test             │
         │   ├─ Build Docker Images      │
         │   └─ Deploy to QA & PROD      │
         └───────────────────────────────┘
                         ↓
         ┌───────────────────────────────┐
         │ GitHub Container Registry     │
         │ ghcr.io/sofioliveto/...       │
         │ ├─ backend:latest             │
         │ └─ frontend:latest            │
         └───────────────────────────────┘
                         ↓
    ┌─────────────────────────────────────────┐
    │        Render.com (Free Tier)           │
    ├─────────────────────────────────────────┤
    │  QA Environment                         │
    │  ├─ palabras-backend-qa.onrender.com    │
    │  └─ palabras-frontend-qa.onrender.com   │
    ├─────────────────────────────────────────┤
    │  PROD Environment (Manual Approval)     │
    │  ├─ palabras-backend-prod.onrender.com  │
    │  └─ palabras-frontend-prod.onrender.com │
    └─────────────────────────────────────────┘
```

## 📋 Requisitos Previos

- [x] Cuenta de GitHub (gratis)
- [x] Cuenta de Render.com (gratis - https://render.com)
- [x] Docker Desktop instalado localmente (opcional, para testing)

## 🚀 Setup Paso a Paso

### Paso 1: Fork/Clone del Repositorio

```bash
# Clonar el repositorio
git clone https://github.com/sofioliveto/TP8-Oliveto.git
cd TP8-Oliveto

# Instalar dependencias (opcional, para desarrollo local)
cd backend && npm install
cd ../frontend && npm install
```

### Paso 2: Configurar GitHub Container Registry

**¡Ya está configurado!** GHCR se habilita automáticamente para todos los repos de GitHub.

**Verificar permisos del workflow**:
1. Ve a Settings → Actions → General
2. En "Workflow permissions", asegúrate que esté seleccionado:
   - ✅ "Read and write permissions"
   - ✅ "Allow GitHub Actions to create and approve pull requests"

**Hacer packages públicos** (para que Render pueda pullearlos gratis):
1. Después del primer push, ve a tu perfil de GitHub
2. Click en "Packages"
3. Para cada package (`backend`, `frontend`):
   - Click en el package → Settings
   - En "Danger Zone" → Change visibility → Public

### Paso 3: Crear Servicios en Render.com

#### 3.1 Crear Backend QA

1. Ir a https://dashboard.render.com → New → Web Service
2. Configurar:
   - **Name**: `palabras-backend-qa`
   - **Environment**: `Docker`
   - **Region**: `Oregon` (o el más cercano gratis)
   - **Branch**: `main`
   - **Image URL**: `ghcr.io/sofioliveto/tp8-oliveto/backend:latest`
     *(Reemplazar `sofioliveto` con tu username de GitHub)*
   - **Port**: `3000`
   - **Plan**: `Free`
3. Environment Variables:
   ```
   ENVIRONMENT_NAME=QA
   NODE_ENV=production
   PORT=3000
   ```
4. Advanced:
   - **Health Check Path**: `/health`
   - **Auto-Deploy**: `Yes` (para deploys automáticos desde GitHub Actions)
5. Click "Create Web Service"
6. **Copiar el Deploy Hook**:
   - Ve a Settings → Deploy Hook
   - Copiar la URL (ej: `https://api.render.com/deploy/srv-...?key=...`)
   - La necesitaremos para GitHub Secrets

#### 3.2 Crear Frontend QA

Repetir el proceso anterior con:
- **Name**: `palabras-frontend-qa`
- **Image URL**: `ghcr.io/sofioliveto/tp8-oliveto/frontend:latest`
- **Port**: `80`
- Resto igual

#### 3.3 Crear Backend PROD

Igual que Backend QA, pero:
- **Name**: `palabras-backend-prod`
- **Environment Variables**:
  ```
  ENVIRONMENT_NAME=PROD
  NODE_ENV=production
  PORT=3000
  ```

#### 3.4 Crear Frontend PROD

Igual que Frontend QA, pero:
- **Name**: `palabras-frontend-prod`
- **Environment Variables**:
  ```
  ENVIRONMENT_NAME=PROD
  ```

### Paso 4: Configurar GitHub Secrets

1. En tu repo de GitHub: Settings → Secrets and variables → Actions → New repository secret

Crear los siguientes secrets:

| Secret Name | Value | Cómo obtenerlo |
|-------------|-------|----------------|
| `RENDER_QA_BACKEND_DEPLOY_HOOK` | `https://api.render.com/deploy/srv-...?key=...` | Render dashboard → palabras-backend-qa → Settings → Deploy Hook |
| `RENDER_QA_FRONTEND_DEPLOY_HOOK` | `https://api.render.com/deploy/srv-...?key=...` | Render dashboard → palabras-frontend-qa → Settings → Deploy Hook |
| `RENDER_QA_BACKEND_URL` | `https://palabras-backend-qa.onrender.com` | URL pública del servicio QA backend |
| `RENDER_QA_FRONTEND_URL` | `https://palabras-frontend-qa.onrender.com` | URL pública del servicio QA frontend |
| `RENDER_PROD_BACKEND_DEPLOY_HOOK` | `https://api.render.com/deploy/srv-...?key=...` | Similar a QA, pero para PROD backend |
| `RENDER_PROD_FRONTEND_DEPLOY_HOOK` | `https://api.render.com/deploy/srv-...?key=...` | Similar a QA, pero para PROD frontend |
| `RENDER_PROD_BACKEND_URL` | `https://palabras-backend-prod.onrender.com` | URL pública del servicio PROD backend |
| `RENDER_PROD_FRONTEND_URL` | `https://palabras-frontend-prod.onrender.com` | URL pública del servicio PROD frontend |

### Paso 5: Configurar GitHub Environments (para approval manual)

1. Settings → Environments → New environment
2. Crear dos environments:

#### Environment: **QA**
- Environment name: `QA`
- No protection rules (deploy automático)

#### Environment: **Production**
- Environment name: `Production`
- ✅ **Required reviewers**: Agregar tu usuario
  - Esto hace que cada deploy a PROD requiera tu aprobación manual
- Wait timer: 0 minutos

### Paso 6: Probar el Pipeline

1. **Hacer un commit y push a `main`**:
   ```bash
   git add .
   git commit -m "Test: Trigger CI/CD pipeline"
   git push origin main
   ```

2. **Monitorear el pipeline**:
   - Ve a Actions → CI/CD with Containers
   - Deberías ver:
     - ✅ Build and test
     - ✅ Build and push images
     - ✅ Deploy to QA
     - ✅ Integration tests on QA
     - ⏸️ Deploy to PROD (esperando tu aprobación)

3. **Aprobar deploy a PROD**:
   - Click en el workflow que está corriendo
   - Verás un banner amarillo "Review deployments"
   - Click → Check "Production" → Approve and deploy

4. **Verificar apps funcionando**:
   - Backend QA: `https://palabras-backend-qa.onrender.com/health`
   - Frontend QA: `https://palabras-frontend-qa.onrender.com`
   - Backend PROD: `https://palabras-backend-prod.onrender.com/health`
   - Frontend PROD: `https://palabras-frontend-prod.onrender.com`

## 🐳 Testing Local con Docker

### Build y Run Backend Localmente

```bash
# Build la imagen
docker build -t palabras-backend:local -f docker/backend/Dockerfile ./backend

# Run el contenedor
docker run -p 3000:3000 \
  -e ENVIRONMENT_NAME=local \
  -e NODE_ENV=development \
  palabras-backend:local

# Verificar health
curl http://localhost:3000/health
```

### Build y Run Frontend Localmente

```bash
# Build la imagen
docker build -t palabras-frontend:local -f docker/frontend/Dockerfile ./frontend

# Run el contenedor
docker run -p 8080:80 palabras-frontend:local

# Abrir en navegador
open http://localhost:8080
```

### Usando Docker Compose (opcional)

Crear `docker-compose.yml`:
```yaml
version: '3.8'
services:
  backend:
    build:
      context: ./backend
      dockerfile: ../docker/backend/Dockerfile
    ports:
      - "3000:3000"
    environment:
      - ENVIRONMENT_NAME=local
      - NODE_ENV=development

  frontend:
    build:
      context: ./frontend
      dockerfile: ../docker/frontend/Dockerfile
    ports:
      - "8080:80"
    depends_on:
      - backend
```

Luego:
```bash
docker-compose up --build
```

## 🔍 Troubleshooting

### Problema: "Package not found" en Render

**Causa**: El package en GHCR es privado y Render no puede pullearlo.

**Solución**:
1. Ve a GitHub → tu perfil → Packages
2. Click en el package → Settings
3. Change visibility → Public

### Problema: "Health check failing" en Render

**Causa**: El servicio tarda en iniciar (cold start en free tier).

**Solución**:
- Esperar 30-60 segundos después del deploy
- Verificar logs en Render dashboard
- Asegurarse que el health check path es `/health`

### Problema: Tests de Cypress fallan en GitHub Actions

**Causa**: QA aún no está listo cuando Cypress corre.

**Solución**: El workflow ya incluye `wait-on` con timeout de 120s. Si sigue fallando, aumentar timeout en `ci-cd-containers.yml`:
```yaml
- name: Wait for QA
  run: npx wait-on ${{ secrets.RENDER_QA_BACKEND_URL }} --timeout 180000
```

### Problema: Workflow falla en "Build and Push Images"

**Causa**: Permisos insuficientes del `GITHUB_TOKEN`.

**Solución**:
1. Settings → Actions → General
2. Workflow permissions → Read and write permissions
3. Re-run el workflow

## 📊 Monitoreo

### Logs en Render
- Dashboard → tu servicio → Logs
- Real-time logs de stdout/stderr
- Buscar por errores o warnings

### Métricas en Render
- Dashboard → tu servicio → Metrics
- CPU usage, Memory usage, Bandwidth

### GitHub Actions Logs
- Actions → workflow run → job → step
- Expandir cada paso para ver output detallado

## 🔄 Rollback en Caso de Error

### Opción 1: Rollback via Render Dashboard
1. Render dashboard → servicio → Manual Deploy
2. Seleccionar una imagen anterior (ej: `v1.0.41-abc123`)
3. Deploy

### Opción 2: Rollback via Git
```bash
# Revertir el último commit
git revert HEAD

# Push (esto triggereará nuevo deploy automático)
git push origin main
```

### Opción 3: Pausar Auto-Deploy
1. Render dashboard → servicio → Settings
2. Auto-Deploy → Off
3. Deployar manualmente la versión correcta

## 💰 Costos y Limitaciones

### Free Tier de Render.com
- ✅ 750 horas/mes por servicio (suficiente para 1 mes 24/7)
- ⚠️ Servicios "duermen" después de 15 min sin tráfico
- ⚠️ Cold start de ~30 segundos al primer request
- ✅ 100GB bandwidth/mes
- ✅ SSL gratis con Let's Encrypt

### Upgrade a Plan Pago (opcional)
Si necesitas evitar el "sleep":
- **Starter**: $7/mes por servicio
  - Siempre activo (no sleep)
  - 512MB RAM garantizada
- **Standard**: $25/mes
  - 2GB RAM
  - Auto-scaling

## 📚 Recursos Adicionales

- [Documentación de Render](https://render.com/docs)
- [GitHub Actions docs](https://docs.github.com/actions)
- [GHCR docs](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Docker best practices](https://docs.docker.com/develop/dev-best-practices/)
- [Documentación técnica completa del TP](./TP8-DOCUMENTACION-TECNICA.md)

## ✅ Checklist de Completitud

- [ ] Dockerfiles creados y optimizados (multi-stage)
- [ ] GitHub Container Registry configurado
- [ ] 4 servicios en Render.com creados (backend/frontend QA/PROD)
- [ ] GitHub Secrets configurados (8 secrets)
- [ ] GitHub Environments configurados (QA, Production)
- [ ] Pipeline ejecutado exitosamente
- [ ] Health checks pasando en QA y PROD
- [ ] Tests de Cypress pasando contra QA
- [ ] Approval manual funcionando para PROD
- [ ] Documentación técnica completada
- [ ] Screenshots de evidencia tomados

## 🎉 ¡Listo!

Tu aplicación ahora tiene:
- ✅ CI/CD automático
- ✅ Deploy a QA en cada push
- ✅ Approval gate antes de PROD
- ✅ Versionado de imágenes Docker
- ✅ Health checks automáticos
- ✅ Testing E2E en QA
- ✅ **Todo por $0/mes** 🎊
