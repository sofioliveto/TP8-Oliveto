# TP8: Contenedores y Automatización - Documentación Técnica

## Tabla de Contenidos
1. [Decisiones Arquitectónicas y Tecnológicas](#decisiones-arquitectónicas-y-tecnológicas)
2. [Implementación](#implementación)
3. [Análisis Comparativo QA vs PROD](#análisis-comparativo-qa-vs-prod)
4. [Reflexión Personal](#reflexión-personal)

---

## Decisiones Arquitectónicas y Tecnológicas

### Stack Tecnológico Elegido

#### Aplicación
- **Backend**: Node.js 20 + Express + SQLite
- **Frontend**: HTML5/CSS3/JavaScript Vanilla
- **Testing**: Jest (unit tests) + Cypress (E2E)

**Justificación**: 
- Stack ya implementado en TPs anteriores (coherencia)
- Node.js tiene excelente soporte de contenedores
- SQLite es embebida (no requiere servicio separado de BD)
- Simplicidad: sin frameworks complejos que dificulten containerización

#### Servicios Cloud Elegidos (100% GRATUITOS)

##### Container Registry: GitHub Container Registry (ghcr.io)

**¿Por qué GitHub Container Registry?**

✅ **Ventajas**:
- **Totalmente gratuito** para repositorios públicos y privados
- **Integración nativa** con GitHub Actions (cero configuración adicional)
- **Ilimitado** en almacenamiento y transferencia para repos públicos
- **Autenticación automática** usando `GITHUB_TOKEN`
- **Versionado robusto** con tags semánticos
- **Visibilidad perfecta** desde el repositorio

❌ **Desventajas consideradas**:
- Menos maduro que Docker Hub (pero suficiente para el TP)
- Sin interfaz web tan rica como ACR o ECR

**Alternativas evaluadas**:
- **Docker Hub**: Límite de 1 repo privado gratis, rate limiting agresivo en pulls
- **Azure Container Registry**: Cuesta ~$5/mes (Basic tier)
- **GitLab Container Registry**: Requeriría migrar todo el proyecto a GitLab

**Decisión**: GHCR es la mejor opción para nuestro caso (proyecto en GitHub, CI/CD con Actions, gratis).

---

##### Hosting: Render.com (QA y PROD)

**¿Por qué Render.com?**

✅ **Ventajas**:
- **Capa gratuita generosa**: 750 horas/mes por servicio
- **Deploy desde registry**: Soporta pull directo de GHCR
- **SSL automático** y gratuito con Let's Encrypt
- **Zero-downtime deploys**
- **Health checks integrados**
- **Logs en tiempo real** accesibles desde dashboard
- **Escalabilidad fácil**: Upgrade a plan pago si es necesario
- **Docker nativo**: No necesita adaptaciones

❌ **Desventaas consideradas**:
- Servicios gratuitos "duermen" después de 15 min de inactividad (cold start de ~30s)
- Menos features enterprise que Azure App Service

**Alternativas evaluadas**:

| Servicio | Ventajas | Desventajas | ¿Por qué NO? |
|----------|----------|-------------|--------------|
| **Railway.app** | $5 crédito mensual gratis | Límite de crédito, luego cobra | Render tiene más horas gratis |
| **Fly.io** | 3 VMs gratis, excelente para geo-replicación | Configuración más compleja | Overkill para este TP |
| **Azure Container Instances** | Integración con Azure DevOps | ~$40/mes si corre 24/7 | **COSTO** |
| **Azure App Services** | Features enterprise | Mínimo $13/mes (B1 Linux) | **COSTO** |
| **Heroku** | Clásico, fácil | Eliminó free tier en 2022 | Ya no es gratis |
| **Google Cloud Run** | Serverless, free tier | Configuración GCP compleja | Render es más simple |

**Decisión**: Render.com ofrece el mejor balance simplicidad/features/costo(gratis) para este TP.

---

##### CI/CD: GitHub Actions

**¿Por qué GitHub Actions?**

✅ **Ventajas**:
- **2000 minutos/mes gratis** para repos privados
- **Ilimitado para repos públicos**
- **Integración nativa** con GHCR (usa `GITHUB_TOKEN` automáticamente)
- **Ecosistema maduro** de actions predefinidas
- **Environments con aprobaciones manuales** (QA → PROD)
- **Secrets management** integrado
- **Ya estábamos usando** Azure DevOps, pero Actions es igual de potente

❌ **Desventajas consideradas**:
- Sintaxis YAML diferente a Azure Pipelines (curva aprendizaje menor)

**Alternativas evaluadas**:
- **Azure DevOps**: Ya lo usamos en TPs anteriores, pero requiere Service Connection a ACR (costo)
- **GitLab CI/CD**: 400 min/mes gratis (menos que Actions), requiere migrar repo
- **CircleCI**: 6000 min/mes gratis, pero configuración más compleja

**Decisión**: GitHub Actions por integración perfecta con GHCR y repo existente.

---

### Decisión QA vs PROD: Mismo Servicio, Diferente Configuración

**¿Por qué usar Render.com tanto para QA como PROD?**

✅ **Ventajas del mismo servicio**:
1. **Consistencia**: Mismo comportamiento en ambos ambientes (elimina "funciona en mi máquina... de QA")
2. **Simplicidad operativa**: Un solo servicio que aprender y mantener
3. **Portabilidad de configs**: Las mismas variables de entorno funcionan igual
4. **Costo $0**: Render permite múltiples servicios gratuitos
5. **Facilita promoción QA→PROD**: Solo cambiar variables, no arquitectura

❌ **Desventajas**:
- Menos "productivo" que tener PROD en servicio enterprise (pero suficiente para el TP)

**¿Cómo se diferencian QA y PROD?**

| Aspecto | QA | PROD |
|---------|-----|------|
| **Recursos** | Shared CPU, 512 MB RAM | Compartidos (free tier) |
| **Variables de entorno** | `ENVIRONMENT_NAME=QA` | `ENVIRONMENT_NAME=PROD` |
| **Deploy automático** | ✅ Automático en cada push a main | ⚠️ Requiere aprobación manual |
| **Health checks** | Interval 30s | Interval 30s |
| **Logging** | Retain 7 días | Retain 7 días (en paid: más) |
| **Monitoreo** | Basic (uptime) | Basic + alertas (en paid) |

**Estrategia de segregación**:
- **Diferentes servicios en Render**: `palabras-backend-qa` vs `palabras-backend-prod`
- **Diferentes URLs**: `palabras-qa.onrender.com` vs `palabras-prod.onrender.com`
- **Diferentes deploy hooks**: Cada uno tiene su webhook único
- **Approval gate en GitHub Actions**: El job `deploy-prod` tiene `environment: Production` que requiere aprobación manual

---

### Configuración de Recursos

#### QA Environment
- **CPU**: Shared (Render free tier)
- **Memoria**: 512 MB
- **Instancias**: 1 (no auto-scaling en free tier)
- **Disco**: Efímero (se resetea en cada deploy)
- **Reinicio automático**: Sí, si crashea

**Justificación**: QA es para testing, no necesita alta disponibilidad ni recursos robustos.

#### PROD Environment
- **CPU**: Shared (Render free tier) - *En producción real: pagaríamos por CPU dedicada*
- **Memoria**: 512 MB - *En producción real: 1-2 GB*
- **Instancias**: 1 - *En producción real: 2+ con load balancing*
- **Disco**: Efímero - *En producción real: persistent storage*
- **Reinicio automático**: Sí

**Justificación**: Para este TP académico, PROD usa la misma config que QA (gratis). En un escenario real con presupuesto, PROD tendría:
- CPU dedicada (no compartida)
- 2+ instancias detrás de load balancer
- Auto-scaling basado en tráfico
- Monitoreo 24/7 con PagerDuty
- Backups automáticos de BD

---

### Gestión de Secretos

**GitHub Actions Secrets (configurados en Settings → Secrets → Actions)**:

| Secret | Descripción | Ejemplo |
|--------|-------------|---------|
| `RENDER_QA_BACKEND_DEPLOY_HOOK` | Webhook para trigger deploy QA backend | `https://api.render.com/deploy/srv-xxx?key=yyy` |
| `RENDER_QA_FRONTEND_DEPLOY_HOOK` | Webhook para trigger deploy QA frontend | `https://api.render.com/deploy/srv-zzz?key=www` |
| `RENDER_QA_BACKEND_URL` | URL pública de backend QA | `https://palabras-backend-qa.onrender.com` |
| `RENDER_QA_FRONTEND_URL` | URL pública de frontend QA | `https://palabras-frontend-qa.onrender.com` |
| `RENDER_PROD_BACKEND_DEPLOY_HOOK` | Webhook para trigger deploy PROD backend | (similar a QA) |
| `RENDER_PROD_FRONTEND_DEPLOY_HOOK` | Webhook para trigger deploy PROD frontend | (similar a QA) |
| `RENDER_PROD_BACKEND_URL` | URL pública de backend PROD | `https://palabras-backend-prod.onrender.com` |
| `RENDER_PROD_FRONTEND_URL` | URL pública de frontend PROD | `https://palabras-frontend-prod.onrender.com` |

**¿Por qué no usamos `GITHUB_TOKEN` para Render?**
- Render no tiene integración OAuth con GitHub para deploys automáticos
- Usamos deploy hooks (webhooks) que son más simples y directos

**Seguridad**:
- ✅ Secrets nunca se exponen en logs
- ✅ Solo accesibles por workflows del repo
- ✅ Pueden rotarse fácilmente desde Render dashboard

---

### Versionado de Imágenes Docker

**Estrategia**: Tags semánticos + SHA

Cada imagen se tagea con:
1. `latest`: Siempre apunta a la última build exitosa de `main`
2. `v1.0.{RUN_NUMBER}-{SHORT_SHA}`: Versión específica trazable

**Ejemplo**:
```
ghcr.io/sofioliveto/tp8-oliveto/backend:latest
ghcr.io/sofioliveto/tp8-oliveto/backend:v1.0.42-a3f8d2c
```

**Ventajas**:
- ✅ **Trazabilidad**: Cada versión tiene SHA del commit exacto
- ✅ **Rollback fácil**: Podemos deployar una versión anterior específica
- ✅ **No usar solo `latest`**: Evita el anti-pattern "latest siempre, sin historial"

**Por qué NO solo `latest`**:
- 🚫 Si algo falla en PROD, no sabemos qué versión funcionó antes
- 🚫 Impossible rollback confiable
- 🚫 Dificulta auditoría ("¿qué cambió entre el lunes y hoy?")

---

## Implementación

### 1. Container Registry (GitHub Container Registry)

#### Configuración

**Paso 1**: El registry está habilitado automáticamente en todos los repos de GitHub

**Paso 2**: Autenticación en el pipeline (GitHub Actions)
```yaml
- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

**Paso 3**: Permisos del `GITHUB_TOKEN`
```yaml
permissions:
  contents: read
  packages: write  # ← Necesario para push a GHCR
```

**Paso 4**: Configurar visibilidad del package (Settings → Packages)
- Opción: Public (para este TP académico)
- Alternativa: Private (para proyectos empresariales)

#### Evidencia

![GHCR con imágenes publicadas](images/ghcr-packages.png)

*Captura mostrando:*
- Backend image con tags `latest` y versionados
- Frontend image con tags `latest` y versionados
- Tamaño de cada imagen (~50MB backend, ~30MB frontend)

---

### 2. Dockerfiles Optimizados

#### Backend Dockerfile

**Características clave**:
- ✅ **Multi-stage build**: Separación build/runtime (reduce tamaño final)
- ✅ **Security**: Non-root user (`nodejs:nodejs`)
- ✅ **Cache layers**: `npm ci` antes de copy código (rebuild más rápido)
- ✅ **Health check**: Endpoint `/health` testeado cada 30s
- ✅ **Production-only deps**: Stage final solo instala `--only=production`

**Optimizaciones**:
```dockerfile
# STAGE 1: Build (con tests)
FROM node:20-alpine AS builder
# ... ejecuta tests para fallar early si hay problemas

# STAGE 2: Runtime (lean)
FROM node:20-alpine AS production
RUN npm ci --only=production  # ← Solo deps de producción
USER nodejs  # ← No corre como root
```

**Tamaño de imagen**:
- Sin optimización: ~300 MB (incluye node_modules completo + devDeps)
- Con optimización: ~50 MB (solo runtime + prod deps)

#### Frontend Dockerfile

**Características clave**:
- ✅ **Nginx alpine**: Base image mínima (~30MB total)
- ✅ **Custom nginx.conf**: Headers de seguridad + gzip + caching
- ✅ **Health check**: Endpoint `/health` custom
- ✅ **Tests en build**: Valida frontend antes de crear imagen

**Configuración Nginx**:
```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;

# Gzip compression
gzip on;
gzip_types text/plain text/css application/json application/javascript;

# Cache static assets
location ~* \.(js|css|png|jpg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

---

### 3. Pipeline CI/CD Completo

#### Arquitectura del Pipeline

```
┌─────────────────────────────────────────────────────────┐
│  JOB 1: build-and-test                                 │
│  ├─ Checkout code                                      │
│  ├─ Setup Node.js 20                                   │
│  ├─ Backend: npm ci → npm test --coverage              │
│  ├─ Verify backend coverage ≥ 70%                      │
│  ├─ Frontend: npm ci → npm test --coverage             │
│  └─ Verify frontend coverage ≥ 70%                     │
└─────────────────────────────────────────────────────────┘
                         ↓
                    (on success)
                         ↓
┌─────────────────────────────────────────────────────────┐
│  JOB 2: build-and-push-images                          │
│  ├─ Generate semantic version tag                      │
│  ├─ Login to GHCR (ghcr.io)                            │
│  ├─ Build backend image (multi-stage)                  │
│  ├─ Push backend image (latest + versioned)            │
│  ├─ Build frontend image (nginx alpine)                │
│  └─ Push frontend image (latest + versioned)           │
└─────────────────────────────────────────────────────────┘
                         ↓
                (only on main branch)
                         ↓
┌─────────────────────────────────────────────────────────┐
│  JOB 3: deploy-qa (Environment: QA)                    │
│  ├─ Trigger Render deploy hook (backend QA)            │
│  ├─ Trigger Render deploy hook (frontend QA)           │
│  ├─ Wait 60s for Render to pull & deploy               │
│  ├─ Health check backend QA (retry 10x)                │
│  └─ Health check frontend QA                           │
└─────────────────────────────────────────────────────────┘
                         ↓
                    (on success)
                         ↓
┌─────────────────────────────────────────────────────────┐
│  JOB 4: integration-tests-qa                           │
│  ├─ Checkout code                                      │
│  ├─ Install Cypress                                    │
│  ├─ Wait for QA to be ready                            │
│  ├─ Run Cypress E2E tests against QA                   │
│  └─ Upload screenshots if tests fail                   │
└─────────────────────────────────────────────────────────┘
                         ↓
            (manual approval required)
                         ↓
┌─────────────────────────────────────────────────────────┐
│  JOB 5: deploy-prod (Environment: Production)          │
│  ├─ 🛑 WAIT FOR MANUAL APPROVAL                        │
│  ├─ Trigger Render deploy hook (backend PROD)          │
│  ├─ Trigger Render deploy hook (frontend PROD)         │
│  ├─ Wait 60s for Render to pull & deploy               │
│  ├─ Health check backend PROD (retry 10x)              │
│  ├─ Health check frontend PROD                         │
│  └─ ✅ Success message with PROD URLs                  │
└─────────────────────────────────────────────────────────┘
```

#### Quality Gates

**Gate 1: Coverage Threshold**
```yaml
# Falla el pipeline si coverage < 70%
if (( $(echo "$AVG < 70" | bc -l) )); then
  echo "❌ Coverage is below 70%"
  exit 1
fi
```

**Gate 2: Integration Tests en QA**
```yaml
# Deploy a PROD bloqueado hasta que Cypress pase en QA
needs: integration-tests-qa
```

**Gate 3: Aprobación Manual**
```yaml
# Requiere click "Approve" en GitHub UI
environment:
  name: Production
```

#### Evidencia del Pipeline

![Pipeline completo ejecutándose](images/github-actions-pipeline.png)

*Captura mostrando:*
- ✅ Build and test: PASSED (15 unit tests)
- ✅ Build images: PUSHED (2 images a GHCR)
- ✅ Deploy QA: DEPLOYED
- ✅ Integration tests: 4/4 Cypress tests PASSED
- ⏸️ Deploy PROD: WAITING FOR APPROVAL

---

### 4. Ambiente QA

#### Configuración en Render.com

**Servicios creados**:
1. **palabras-backend-qa**
   - Type: Web Service
   - Environment: Docker
   - Image URL: `ghcr.io/sofioliveto/tp8-oliveto/backend:latest`
   - Port: 3000
   - Health Check Path: `/health`

2. **palabras-frontend-qa**
   - Type: Web Service
   - Environment: Docker
   - Image URL: `ghcr.io/sofioliveto/tp8-oliveto/frontend:latest`
   - Port: 80
   - Health Check Path: `/health`

**Variables de Entorno (QA)**:
```
ENVIRONMENT_NAME=QA
NODE_ENV=production
PORT=3000
DB_PATH=/app/palabras.db
```

**Deploy Hook**:
```
https://api.render.com/deploy/srv-abc123?key=xyz456
```

#### Evidencia QA

![Render dashboard - QA services](images/render-qa-dashboard.png)

*Captura mostrando:*
- Backend QA: Status "Live" 🟢
- Frontend QA: Status "Live" 🟢
- Last deploy: "2 minutes ago"
- Health check: Passing ✅

![App funcionando en QA](images/qa-app-screenshot.png)

*Captura de la aplicación corriendo en `https://palabras-backend-qa.onrender.com`:*
- Lista de palabras cargando correctamente
- Formulario funcionando (agregar/eliminar palabras)
- Health endpoint respondiendo: `{"status":"OK","environment":"QA"}`

---

### 5. Ambiente PROD

#### Configuración en Render.com

**Servicios creados** (similar a QA, diferentes recursos):
1. **palabras-backend-prod**
   - Type: Web Service
   - Environment: Docker
   - Image URL: `ghcr.io/sofioliveto/tp8-oliveto/backend:latest`
   - Port: 3000
   - Health Check Path: `/health`
   - *En plan pago: Auto-scaling enabled, 2+ instances*

2. **palabras-frontend-prod**
   - Type: Web Service
   - Environment: Docker
   - Image URL: `ghcr.io/sofioliveto/tp8-oliveto/frontend:latest`
   - Port: 80
   - Health Check Path: `/health`

**Variables de Entorno (PROD)**:
```
ENVIRONMENT_NAME=PROD
NODE_ENV=production
PORT=3000
DB_PATH=/app/palabras.db
```

#### Continuous Deployment

**Estrategia**:
1. Push a `main` → CI tests → Build images
2. Auto-deploy a QA → Integration tests
3. **Manual approval gate** ← Alguien del equipo debe aprobar
4. Deploy a PROD

**Por qué approval manual**:
- 🛡️ Seguridad: Evita deploys accidentales a PROD
- 👁️ Review: Permite verificar QA antes de promover
- 📋 Compliance: Muchas empresas requieren "change approval" para PROD

#### Diferencias Clave QA → PROD

| Aspecto | QA | PROD | Justificación |
|---------|-----|------|---------------|
| **Deploy automático** | ✅ Sí (cada push a main) | ⏸️ No (requiere aprobación) | PROD no debe cambiar sin validación humana |
| **Testing previo** | Unit tests solamente | Unit + Integration en QA | PROD solo recibe código probado en QA |
| **Variables de entorno** | `ENVIRONMENT_NAME=QA` | `ENVIRONMENT_NAME=PROD` | Para diferenciar logs y comportamiento |
| **Monitoreo** | Basic (uptime) | Enhanced (uptime + alertas) | PROD necesita alertas 24/7 |
| **Recursos** (free tier) | 512MB RAM, shared CPU | 512MB RAM, shared CPU | En plan pago: PROD tendría 2GB, CPU dedicada |
| **Auto-scaling** (free tier) | No | No | En plan pago: PROD tendría 2-5 instancias según carga |
| **Backups BD** (free tier) | No (SQLite efímero) | No | En plan pago: Daily backups a S3 |
| **URL** | `palabras-backend-qa.onrender.com` | `palabras-backend-prod.onrender.com` | Segregación clara |

#### Evidencia PROD

![Render dashboard - PROD services](images/render-prod-dashboard.png)

![App funcionando en PROD](images/prod-app-screenshot.png)

*Captura de la aplicación corriendo en `https://palabras-backend-prod.onrender.com`:*
- Health endpoint: `{"status":"OK","environment":"PROD"}`
- Same app funcionando, pero con datos de producción
- SSL cert válido (Let's Encrypt)

---

## Análisis Comparativo QA vs PROD

### Tabla Comparativa Detallada

| Aspecto | QA | PROD | Justificación |
|---------|-----|------|---------------|
| **Servicio usado** | Render.com Web Service (Docker) | Render.com Web Service (Docker) | Mismo servicio para consistencia operativa |
| **CPU** | Shared (free tier) | Shared (free tier) | *En escenario real: PROD tendría CPU dedicada para performance predecible* |
| **Memoria** | 512 MB | 512 MB | *En escenario real: PROD tendría 1-2GB para soportar más tráfico* |
| **Número de instancias** | 1 (no replicas) | 1 | *En escenario real: PROD tendría 2+ instancias con load balancer* |
| **Escalabilidad** | Manual (free tier) | Manual | *En escenario real: PROD tendría auto-scaling 1-10 instancias* |
| **Costos** | $0/mes (free tier) | $0/mes (free tier) | *En escenario real: QA ~$7/mes, PROD ~$25/mes* |
| **Monitoreo/Logs** | Basic uptime check, logs 7 días | Basic uptime check, logs 7 días | *En escenario real: PROD tendría Datadog/New Relic con alerting* |
| **Deploy automático** | ✅ Automático en push a main | ⏸️ Requiere aprobación manual | Evitar deploys accidentales a PROD |
| **Testing previo** | Solo unit tests | Unit tests + E2E en QA | PROD recibe código pre-validado |
| **Database** | SQLite efímero (se resetea en deploy) | SQLite efímero | *En escenario real: PostgreSQL manejado con backups* |
| **SSL/TLS** | ✅ Gratis (Let's Encrypt) | ✅ Gratis (Let's Encrypt) | Render provee SSL automático |
| **Health checks** | Interval 30s, timeout 3s | Interval 30s, timeout 3s | Igual configuración para ambos |
| **Zero-downtime deploy** | ✅ Sí (Render feature) | ✅ Sí (Render feature) | Render hace rolling updates automáticos |

---

### Ventajas de Usar el Mismo Servicio

✅ **Consistencia total**:
- Mismo comportamiento de runtime
- Mismas limitaciones y quirks
- Elimina "funciona en QA pero falla en PROD por diferencias de plataforma"

✅ **Operacionalmente simple**:
- Un solo dashboard que monitorear (Render)
- Mismas herramientas de debugging
- Conocimiento transferible 1:1 entre ambientes

✅ **Portabilidad de configuración**:
- Same Dockerfile funciona idéntico
- Variables de entorno iguales (salvo `ENVIRONMENT_NAME`)
- Secrets manejados de la misma manera

✅ **Costo-eficiencia**:
- Render permite múltiples servicios gratuitos
- Aprender un servicio vs. aprender dos

✅ **Facilita promoción QA→PROD**:
- No hay "migración" de servicio, solo cambio de variables
- Rollback es simétrico

---

### Desventajas de Usar el Mismo Servicio

❌ **Limitaciones del free tier afectan a PROD**:
- 750 hrs/mes → servicios "duermen" si no hay tráfico (cold start ~30s)
- No auto-scaling en free tier
- Sin recursos dedicados (shared CPU)

**Mitigación**: En un proyecto real con presupuesto, PROD tendría plan de pago ($7-25/mes)

❌ **Menos features enterprise en PROD**:
- Sin SLA garantizado (Render free tier es "best effort")
- Sin soporte prioritario
- Sin disaster recovery automático

**Mitigación**: Evaluar si la aplicación necesita SLA 99.9% (en cuyo caso, migrar PROD a AWS/Azure)

❌ **Configuración idéntica puede ser limitante**:
- Si PROD necesita 10GB RAM y QA con 512MB está OK, tendrías servicios muy diferentes

**Mitigación**: En Render, esto se resuelve fácil (QA en free tier, PROD en plan Pro)

---

### Análisis de Alternativas

#### ¿Por qué NO usar servicios diferentes para QA y PROD?

**Opción descartada: QA en Render (gratis) + PROD en Azure App Service ($13/mes)**

❌ Contras:
- Complejidad: Aprender dos plataformas diferentes
- Inconsistencias: Variables de entorno diferentes, comportamiento runtime diferente
- Riesgo: "Funciona en Render/QA pero falla en Azure/PROD"
- Costos: No necesario para TP académico

✅ Pros (cuando tiene sentido):
- PROD en Azure da features enterprise (SLA 99.95%, soporte 24/7, integración con otros servicios Azure)
- QA en servicio barato/gratis para ahorrar

**¿Cuándo usaría servicios diferentes?**:
- Si la empresa ya tiene infraestructura en Azure y PROD debe estar ahí por compliance
- Si PROD necesita features que Render no ofrece (ej: integración con Azure AD)
- Si el presupuesto permite pagar Azure PROD pero no QA (entonces QA va a Render gratis)

---

#### ¿Por qué NO migrar a Kubernetes?

**Opción descartada: AKS (Azure Kubernetes Service) o GKE**

❌ Contras:
- **Complejidad masiva**: Requiere conocer Deployments, Services, Ingress, ConfigMaps, Secrets, Helm
- **Overkill para esta app**: 2 servicios simples no necesitan orquestación K8s
- **Costos**: AKS cluster mínimo ~$70/mes (1 node)
- **Overhead operativo**: Mantenimiento de cluster, actualizaciones de K8s, troubleshooting complejo

✅ Pros (cuando tiene sentido):
- Si la app crece a 50+ microservicios
- Si necesitamos auto-scaling horizontal sofisticado
- Si tráfico es 10,000+ RPS
- Si equipo tiene expertise en K8s

**Decisión**: Para TP8, Render es más apropiado. K8s sería justificado si escaláramos 10x.

---

### Análisis de Costos

#### Costos Estimados por Ambiente

| Concepto | QA (Actual) | PROD (Actual) | QA (Plan Pago) | PROD (Plan Pago) |
|----------|-------------|---------------|----------------|------------------|
| **Web Service Backend** | $0 (free tier) | $0 (free tier) | $7/mes (Starter) | $25/mes (Standard) |
| **Web Service Frontend** | $0 (free tier) | $0 (free tier) | $7/mes (Starter) | $25/mes (Standard) |
| **Container Registry** | $0 (GHCR gratis) | $0 (GHCR gratis) | $0 | $0 |
| **CI/CD (GitHub Actions)** | $0 (repo público) | $0 (repo público) | $0 | $0 |
| **Database** | $0 (SQLite) | $0 (SQLite) | $7/mes (PostgreSQL) | $15/mes (PostgreSQL) |
| **Total/mes** | **$0** | **$0** | **$21** | **$65** |

#### Comparación con Alternativas

**Azure Stack** (lo que sugiere la guía):
- ACR Basic: $5/mes
- ACI backend (1 vCPU, 1.5GB): ~$40/mes
- ACI frontend: ~$40/mes
- Azure DevOps: $0 (free tier)
- **Total QA+PROD**: ~$170/mes 💸

**AWS Stack**:
- ECR: $0.10/GB storage
- App Runner backend: ~$25/mes
- App Runner frontend: ~$25/mes
- **Total**: ~$55/mes

**Nuestra Stack (Render + GHCR)**:
- **Free tier**: $0/mes ✅
- **Paid tier**: $65/mes (PROD) + $21/mes (QA) = $86/mes

**Conclusión**: Render es 50% más barato que Azure y 100% gratis en free tier.

---

### Escalabilidad a Futuro

#### ¿Cuándo migraríamos a Kubernetes?

**Señales de que es hora de K8s**:
1. **Número de servicios**: Cuando pasamos de 2-3 servicios a 20+ microservicios
2. **Complejidad de deploys**: Cuando necesitamos canary deployments, blue-green, etc.
3. **Auto-scaling avanzado**: Cuando necesitamos escalar basado en métricas custom (no solo CPU/memoria)
4. **Multi-cloud**: Cuando queremos portabilidad total entre AWS/Azure/GCP
5. **Team size**: Cuando tenemos equipo dedicado de DevOps/SRE

**Para nuestra app actual**: NO tiene sentido K8s. Render/Railway/Fly.io son perfectos.

#### ¿Qué cambiaría si la app crece 10x?

**Escenario**: De 10 usuarios/día a 10,000 usuarios/día

**Cambios necesarios**:

1. **Base de Datos**:
   - ❌ SQLite (single file, locks en writes)
   - ✅ PostgreSQL managed (ej: Railway PostgreSQL, Render PostgreSQL, o Supabase)
   - ✅ Connection pooling (PgBouncer)
   - ✅ Read replicas para queries pesadas

2. **Backend**:
   - ✅ Mínimo 2 instancias con load balancer
   - ✅ Auto-scaling 2-10 instancias (Render lo soporta en plan Pro)
   - ✅ Redis para caching (reducir carga a DB)
   - ✅ CDN para assets estáticos (Cloudflare gratis)

3. **Frontend**:
   - ✅ Servir desde CDN en lugar de Nginx (Vercel/Netlify gratis con CDN global)
   - ✅ Lazy loading de componentes
   - ✅ Service worker para offline

4. **Monitoring**:
   - ✅ APM (Application Performance Monitoring) con Datadog o New Relic
   - ✅ Error tracking con Sentry
   - ✅ Uptime monitoring con UptimeRobot
   - ✅ Alertas a Slack/PagerDuty

5. **CI/CD**:
   - ✅ Quality gates más estrictos (performance budgets, security scans)
   - ✅ Canary deployments (5% de tráfico → nueva versión, si OK → 100%)

**Costo estimado a 10x escala**: ~$200-300/mes (aún mucho más barato que Azure)

---

## Reflexión Personal

### Desafíos Encontrados

#### 1. **Docker multi-stage builds con tests**

**Problema**: El Dockerfile ejecuta `npm test` en el stage builder, pero si hay archivos faltantes (ej: `__tests__`), falla el build.

**Solución**: Asegurar que `.dockerignore` NO excluye `__tests__` en el build stage. Solo el stage final (production) excluye devDeps.

**Aprendizaje**: Multi-stage builds son poderosas pero requieren pensar "qué se copia en cada stage".

---

#### 2. **Render free tier y cold starts**

**Problema**: Los servicios gratuitos de Render "duermen" después de 15 min sin tráfico. El primer request después tiene un cold start de ~30 segundos.

**Impacto**: En QA, no es problema. En PROD, podría ser frustrante para usuarios.

**Soluciones evaluadas**:
- ❌ Ping service cada 10 min (Render lo detecta y lo bloquea)
- ❌ Upgrade a plan pago (cuesta $7/mes, pero fuera del alcance del TP "sin gastar")
- ✅ Documentar la limitación y vivir con ella (este es un TP académico, no producción real)

**Aprendizaje**: "Free tier" siempre tiene trade-offs. Entender cuáles son antes de comprometerse.

---

#### 3. **GitHub Container Registry y visibilidad**

**Problema**: Inicialmente, las imágenes pusheadas a GHCR eran privadas por defecto, y Render no podía pullearlas (requería autenticación).

**Solución**: Cambiar visibilidad del package a "Public" en GitHub Settings → Packages.

**Aprendizaje**: GHCR tiene permisos granulares. Para proyectos open source, marcarlas públicas desde el inicio.

---

#### 4. **Deploy hooks vs. Render API**

**Problema**: Render tiene una API completa, pero también "deploy hooks" (webhooks simples). ¿Cuál usar?

**Decisión**: Deploy hooks son más simples (solo un `curl -X POST`), mientras que la API requiere bearer token + endpoints complejos.

**Aprendizaje**: Para este TP, deploy hooks son suficientes. La API sería útil si necesitáramos inspeccionar estado del deploy o hacer rollbacks programáticos.

---

### Qué Mejoraría en una Implementación Productiva Real

#### 1. **Base de Datos**

**Actual**: SQLite embebida (se resetea en cada deploy)

**Mejora**: PostgreSQL manejado con:
- Backups automáticos diarios
- Point-in-time recovery
- Read replicas para scaling
- Connection pooling

**Por qué**: SQLite es perfecta para desarrollo, pero en PROD con múltiples instancias, necesitamos DB centralizada y persistente.

---

#### 2. **Secrets Management**

**Actual**: GitHub Secrets (bien, pero básico)

**Mejora**: HashiCorp Vault o AWS Secrets Manager
- Rotación automática de credenciales
- Audit log de quién accedió a qué secret
- Encryption at rest + in transit

**Por qué**: En empresas grandes, compliance requiere secrets management robusto.

---

#### 3. **Observability**

**Actual**: Logs básicos de Render + health checks

**Mejora**: Stack completo de observability:
- **Metrics**: Prometheus + Grafana (CPU, memoria, latencia, RPS)
- **Logs**: ELK Stack o Loki (búsqueda avanzada, alertas)
- **Traces**: OpenTelemetry + Jaeger (distributed tracing)
- **Alerting**: PagerDuty (notificación a on-call cuando PROD falla)

**Por qué**: Para debuggear problemas en producción, necesitamos visibilidad completa.

---

#### 4. **CI/CD Avanzado**

**Actual**: Pipeline lineal (build → QA → approve → PROD)

**Mejora**:
- **Canary deployments**: Deploy gradual (5% → 25% → 100%)
- **Blue-green deployments**: Tener dos versiones de PROD, switchear tráfico instantáneamente
- **Automated rollback**: Si error rate aumenta, rollback automático
- **Performance testing**: Gatling/K6 corriendo en cada deploy a QA

**Por qué**: Reducir riesgo de downtime en PROD.

---

#### 5. **Security**

**Actual**: Imágenes construidas sin escaneo de vulnerabilidades

**Mejora**:
- **Trivy** o **Snyk** escaneando cada imagen antes de push
- **Dependabot** actualizando dependencias automáticamente
- **OWASP ZAP** para security testing de APIs
- **WAF** (Web Application Firewall) delante de PROD

**Por qué**: Evitar que vulnerabilidades conocidas lleguen a producción.

---

### Aprendizajes Clave del Trabajo Práctico

#### 1. **Contenedores no son magia, son herramientas**

Antes del TP: "Docker es complicado y solo para DevOps"

Después del TP: "Docker es un packaging estándar que hace deployments predecibles"

**Insight**: El valor de Docker no es la tecnología en sí, sino la **consistencia** que aporta ("funciona en mi máquina" = "funciona en PROD").

---

#### 2. **Multi-stage builds son game-changers**

Reducir imagen de 300MB a 50MB no solo ahorra disco:
- ✅ Deploys 6x más rápidos (menos tiempo descargando imagen)
- ✅ Menos superficie de ataque (menos paquetes = menos vulnerabilidades)
- ✅ Costos más bajos (menos bandwidth, menos storage)

**Insight**: Optimización de imágenes no es "nice to have", es fundamental.

---

#### 3. **Quality gates previenen desastres**

El approval manual antes de PROD nos salvó 2 veces durante el desarrollo:
1. Un test de Cypress estaba fallando intermitentemente (flaky test)
2. Una migración de BD faltaba en el script de startup

**Insight**: La fricción del approval manual es feature, no bug. Previene deployar código roto.

---

#### 4. **Free tier es viable para MVPs**

Stack completo por $0/mes:
- Container registry
- Hosting QA + PROD
- CI/CD
- SSL

**Insight**: En 2024, es posible lanzar un producto a producción sin gastar un centavo en infra. Los límites del free tier solo importan cuando tienes usuarios (buen problema).

---

#### 5. **Documentación > Código**

Este documento me forzó a **justificar cada decisión**. Eso me hizo:
- Cuestionarme si Render era realmente mejor que Railway
- Evaluar alternativas que no había considerado (Fly.io, Cloud Run)
- Entender trade-offs en lugar de elegir "lo primero que funciona"

**Insight**: Escribir documentación técnica no es "trabajo extra", es pensamiento crítico materializado.

---

### Comparación con Otros Stacks Evaluados

#### Opción 1: Azure Stack (guía oficial)

**Stack**: ACR + ACI + Azure Pipelines

✅ **Ventajas**:
- Integración perfecta en ecosistema Azure
- Features enterprise (SLA, soporte 24/7)
- Ya estábamos usando Azure DevOps

❌ **Desventajas**:
- **Costo**: ~$170/mes para QA+PROD
- Complejidad: Requiere Service Connections, Resource Groups, CLI complejo

**¿Por qué NO elegí esto?**: Requisito del TP era "no gastar nada". Azure no tiene free tier suficiente.

---

#### Opción 2: AWS Stack

**Stack**: ECR + App Runner + GitHub Actions

✅ **Ventajas**:
- Free tier decente (12 meses)
- App Runner es excelente para contenedores sin K8s

❌ **Desventajas**:
- Costo después de 12 meses: ~$55/mes
- Curva de aprendizaje de AWS (IAM, VPC, etc.)

**¿Por qué NO elegí esto?**: Render es más simple y permanentemente gratis.

---

#### Opción 3: GitLab Stack

**Stack**: GitLab Container Registry + GitLab CI/CD + Cloud Run

✅ **Ventajas**:
- GitLab CI/CD es muy potente
- 400 minutos/mes gratis
- Cloud Run tiene free tier generoso

❌ **Desventajas**:
- Requeriría migrar repo de GitHub a GitLab
- 400 min/mes es menos que GitHub Actions (2000 min)

**¿Por qué NO elegí esto?**: Ya tenía el repo en GitHub, migrar no aportaba valor.

---

### Conclusión Final

Este TP me demostró que **containers + CI/CD no son solo para empresas grandes**. Con las herramientas correctas (GitHub Actions + Render + GHCR), cualquier developer puede:

- Deployar a producción en minutos
- Tener ambientes QA y PROD profesionales
- Implementar quality gates robustos
- Todo por **$0/mes**

El verdadero aprendizaje no fue "cómo usar Docker" sino **cómo tomar decisiones arquitectónicas justificadas**:
- Evaluar alternativas con criterio
- Entender trade-offs (costos vs features vs complejidad)
- Documentar decisiones para el "yo del futuro"

**Si tuviera que hacer este TP de nuevo**, elegiría el mismo stack. Render + GHCR + GitHub Actions es el sweet spot de simplicidad, costo, y features para proyectos de este tamaño.
