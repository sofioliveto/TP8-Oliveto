# TP8: Lista de Verificación y Pasos a Seguir

## ✅ Estado Actual

### Lo que YA está hecho (por el agente):
- [x] Dockerfiles optimizados creados
  - [x] Backend: Multi-stage build (50MB)
  - [x] Frontend: Nginx alpine (30MB)
- [x] .dockerignore para backend y frontend
- [x] Pipeline CI/CD completo en GitHub Actions
- [x] Documentación técnica exhaustiva (TP8-DOCUMENTACION-TECNICA.md)
- [x] Guía de setup paso a paso (TP8-SETUP-GUIDE.md)
- [x] Resumen ejecutivo (TP8-RESUMEN.md)
- [x] Código committed y pusheado a GitHub

---

## 🔧 Lo que FALTA hacer (por vos, Sofía):

### Paso 1: Revisar y Entender la Implementación (30 min)

- [ ] Leer `TP8-RESUMEN.md` completo
- [ ] Leer `TP8-SETUP-GUIDE.md` para entender el setup
- [ ] Revisar `TP8-DOCUMENTACION-TECNICA.md` (al menos las secciones principales)
- [ ] Entender el flujo del pipeline en `.github/workflows/ci-cd-containers.yml`
- [ ] Revisar los Dockerfiles en `docker/backend/` y `docker/frontend/`

**Objetivo**: Entender qué se implementó y por qué.

---

### Paso 2: Configurar GitHub (10 min)

#### A. Habilitar permisos de GitHub Actions
- [ ] Ir a: Settings → Actions → General
- [ ] En "Workflow permissions":
  - [ ] Seleccionar "Read and write permissions"
  - [ ] Marcar "Allow GitHub Actions to create and approve pull requests"
- [ ] Click "Save"

#### B. Crear GitHub Environments
- [ ] Ir a: Settings → Environments → New environment
- [ ] Crear environment "QA":
  - Name: `QA`
  - No protection rules (dejar vacío)
  - Click "Configure environment"
- [ ] Crear environment "Production":
  - Name: `Production`
  - ✅ Marcar "Required reviewers"
  - Agregar tu usuario como reviewer
  - Wait timer: 0 minutos
  - Click "Save protection rules"

**Objetivo**: Permitir que el pipeline funcione y requiera aprobación para PROD.

---

### Paso 3: Crear Cuenta en Render.com (5 min)

- [ ] Ir a https://render.com
- [ ] Click "Get Started" → Sign up with GitHub
- [ ] Autorizar Render a acceder a tu GitHub
- [ ] Completar perfil (si lo pide)
- [ ] Verificar email (si lo pide)

**Objetivo**: Tener acceso a Render dashboard.

---

### Paso 4: Crear Servicios en Render (4 servicios × 5 min = 20 min)

#### Servicio 1: Backend QA

- [ ] En Render dashboard: New → Web Service
- [ ] Configurar:
  - **Name**: `palabras-backend-qa`
  - **Region**: Oregon (o el más cercano gratis)
  - **Environment**: Docker
  - **Image URL**: `ghcr.io/sofioliveto/tp8-oliveto/backend:latest`
    *(Reemplazar `sofioliveto` con tu username de GitHub)*
  - **Port**: `3000`
  - **Instance Type**: Free
- [ ] Advanced Settings:
  - **Health Check Path**: `/health`
  - **Auto-Deploy**: Yes
- [ ] Environment Variables (Add):
  ```
  ENVIRONMENT_NAME=QA
  NODE_ENV=production
  PORT=3000
  ```
- [ ] Click "Create Web Service"
- [ ] **Esperar** a que el servicio se despliegue (tarda ~2-3 min la primera vez)
- [ ] **Copiar la URL** (ej: `https://palabras-backend-qa.onrender.com`)
- [ ] Ir a Settings → Deploy Hook
- [ ] **Copiar el Deploy Hook** (ej: `https://api.render.com/deploy/srv-xxx?key=yyy`)

**Anotar**:
- URL Backend QA: `________________________`
- Deploy Hook Backend QA: `________________________`

#### Servicio 2: Frontend QA

- [ ] Repetir proceso anterior con:
  - **Name**: `palabras-frontend-qa`
  - **Image URL**: `ghcr.io/sofioliveto/tp8-oliveto/frontend:latest`
  - **Port**: `80`
  - **Environment Variables**:
    ```
    ENVIRONMENT_NAME=QA
    ```
  - Resto igual

**Anotar**:
- URL Frontend QA: `________________________`
- Deploy Hook Frontend QA: `________________________`

#### Servicio 3: Backend PROD

- [ ] Repetir proceso de Backend QA pero:
  - **Name**: `palabras-backend-prod`
  - **Environment Variables**:
    ```
    ENVIRONMENT_NAME=PROD
    NODE_ENV=production
    PORT=3000
    ```

**Anotar**:
- URL Backend PROD: `________________________`
- Deploy Hook Backend PROD: `________________________`

#### Servicio 4: Frontend PROD

- [ ] Repetir proceso de Frontend QA pero:
  - **Name**: `palabras-frontend-prod`
  - **Environment Variables**:
    ```
    ENVIRONMENT_NAME=PROD
    ```

**Anotar**:
- URL Frontend PROD: `________________________`
- Deploy Hook Frontend PROD: `________________________`

---

### Paso 5: Configurar GitHub Secrets (10 min)

- [ ] En tu repo de GitHub: Settings → Secrets and variables → Actions
- [ ] New repository secret (repetir 8 veces):

| Secret Name | Value (usar tus URLs/hooks anotados arriba) |
|-------------|---------------------------------------------|
| `RENDER_QA_BACKEND_DEPLOY_HOOK` | _(Deploy Hook Backend QA)_ |
| `RENDER_QA_FRONTEND_DEPLOY_HOOK` | _(Deploy Hook Frontend QA)_ |
| `RENDER_QA_BACKEND_URL` | _(URL Backend QA)_ |
| `RENDER_QA_FRONTEND_URL` | _(URL Frontend QA)_ |
| `RENDER_PROD_BACKEND_DEPLOY_HOOK` | _(Deploy Hook Backend PROD)_ |
| `RENDER_PROD_FRONTEND_DEPLOY_HOOK` | _(Deploy Hook Frontend PROD)_ |
| `RENDER_PROD_BACKEND_URL` | _(URL Backend PROD)_ |
| `RENDER_PROD_FRONTEND_URL` | _(URL Frontend PROD)_ |

**Verificar**:
- [ ] 8 secrets creados correctamente
- [ ] Nombres exactos (case-sensitive)

---

### Paso 6: Hacer Primer Deploy (15 min)

#### A. Mergear la PR
- [ ] Ir a tu repo → Pull requests → PR de `copilot/implement-automatic-containers`
- [ ] Revisar los cambios
- [ ] Click "Merge pull request" → "Confirm merge"

#### B. Monitorear el Pipeline
- [ ] Ir a Actions → "CI/CD with Containers"
- [ ] Debería aparecer un workflow corriendo automáticamente
- [ ] Monitorear cada job:
  - [ ] ✅ Build and Test (debería pasar)
  - [ ] ✅ Build and Push Images (debería pasar)
  - [ ] ✅ Deploy QA (debería pasar)
  - [ ] ✅ Integration Tests (debería pasar)
  - [ ] ⏸️ Deploy PROD (esperando aprobación)

#### C. Aprobar Deploy a PROD
- [ ] Cuando veas el banner "Review deployments"
- [ ] Click en "Review deployments"
- [ ] Marcar "Production"
- [ ] Click "Approve and deploy"
- [ ] Esperar a que termine el job "Deploy PROD"

#### D. Verificar que todo funciona
- [ ] Abrir Backend QA en browser: `https://palabras-backend-qa.onrender.com/health`
  - Debería mostrar: `{"status":"OK","environment":"QA"}`
- [ ] Abrir Frontend QA: `https://palabras-frontend-qa.onrender.com`
  - Debería mostrar la app funcionando
  - Probar agregar/eliminar palabras
- [ ] Abrir Backend PROD: `https://palabras-backend-prod.onrender.com/health`
  - Debería mostrar: `{"status":"OK","environment":"PROD"}`
- [ ] Abrir Frontend PROD: `https://palabras-frontend-prod.onrender.com`
  - Debería mostrar la app funcionando

---

### Paso 7: Hacer Imágenes de GHCR Públicas (5 min)

#### Problema:
Por defecto, las imágenes pusheadas a GHCR son privadas, y Render no puede pullearlas sin autenticación.

#### Solución:
- [ ] Ir a tu perfil de GitHub → Packages
- [ ] Click en package "backend" → Settings
- [ ] Scroll hasta "Danger Zone"
- [ ] Click "Change visibility" → Public → Confirmar
- [ ] Repetir para package "frontend"

**Verificar**:
- [ ] Ambos packages son "Public"
- [ ] Re-deployar en Render si falló el primer intento

---

### Paso 8: Tomar Evidencia para el TP (30 min)

Crear carpeta `images/tp8/` en tu repo y capturar:

#### Screenshots de GHCR:
- [ ] Packages listados (backend + frontend)
- [ ] Tags de cada imagen (latest + versionados)
- [ ] Detalles de una imagen (tamaño, SHA, etc.)

#### Screenshots de GitHub Actions:
- [ ] Pipeline completo ejecutándose (overview)
- [ ] Job "Build and Test" exitoso (con tests pasando)
- [ ] Job "Build and Push Images" exitoso
- [ ] Job "Deploy QA" exitoso (con health checks)
- [ ] Job "Integration Tests" exitoso (Cypress)
- [ ] Job "Deploy PROD" con approval pendiente
- [ ] Job "Deploy PROD" después de aprobar

#### Screenshots de Render:
- [ ] Dashboard con los 4 servicios "Live"
- [ ] Logs de backend QA mostrando requests
- [ ] Métricas de un servicio (CPU, memoria)

#### Screenshots de la App funcionando:
- [ ] Backend QA `/health` endpoint
- [ ] Frontend QA con lista de palabras
- [ ] Agregar una palabra (antes/después)
- [ ] Backend PROD `/health` endpoint
- [ ] Frontend PROD funcionando

#### Diagrama de arquitectura:
- [ ] Copiar el diagrama de `TP8-RESUMEN.md`
- [ ] Opcionalmente: crear versión visual con draw.io

---

### Paso 9: Completar la Documentación (1 hora)

- [ ] Revisar `TP8-DOCUMENTACION-TECNICA.md` completo
- [ ] Agregar sección "Evidencia" con screenshots
- [ ] Personalizar la "Reflexión Personal" con tu experiencia
- [ ] Revisar que todas las URLs estén correctas (reemplazar placeholders)
- [ ] Agregar cualquier desafío adicional que encontraste

---

### Paso 10: Preparar Defensa Oral (2 horas)

#### Estudiar las 29 preguntas del TP:
- [ ] Leer cada pregunta en `TP8-DOCUMENTACION-TECNICA.md`
- [ ] Practicar respuestas en voz alta
- [ ] Asegurarte de poder justificar cada decisión

#### Preguntas clave para practicar:
1. ¿Por qué elegiste Render en lugar de Azure?
2. ¿Por qué usar el mismo servicio para QA y PROD?
3. ¿Cómo funciona el multi-stage build en Docker?
4. ¿Qué es un quality gate y cuáles implementaste?
5. ¿Cómo harías un rollback si falla un deploy a PROD?
6. ¿Cuándo migrarías a Kubernetes?
7. ¿Qué optimizaciones hiciste en los Dockerfiles?
8. ¿Cómo se diferencian QA y PROD en tu arquitectura?

#### Preparar demo en vivo:
- [ ] Mostrar pipeline ejecutándose
- [ ] Hacer un cambio pequeño (ej: editar README)
- [ ] Push → mostrar deploy automático a QA
- [ ] Mostrar approval gate
- [ ] Aprobar y mostrar deploy a PROD
- [ ] Mostrar app funcionando en ambos ambientes

---

## 📊 Checklist Final Antes de Entregar

### Repositorio:
- [ ] Todos los archivos committed
- [ ] PR mergeada a `main`
- [ ] README actualizado con instrucciones
- [ ] Sin archivos sensibles (secrets, passwords)
- [ ] .gitignore correcto

### Servicios Externos:
- [ ] 4 servicios en Render funcionando (status "Live")
- [ ] Imágenes en GHCR públicas y accesibles
- [ ] GitHub Secrets configurados (8 secrets)
- [ ] GitHub Environments configurados (QA + Production)

### Pipeline:
- [ ] Pipeline ejecutándose exitosamente
- [ ] Todos los jobs pasando (verde)
- [ ] Coverage ≥70% en backend y frontend
- [ ] Tests de Cypress pasando

### Aplicaciones:
- [ ] Backend QA funcionando (`/health` responde)
- [ ] Frontend QA funcionando (UI carga)
- [ ] Backend PROD funcionando
- [ ] Frontend PROD funcionando

### Documentación:
- [ ] TP8-DOCUMENTACION-TECNICA.md completo
- [ ] TP8-SETUP-GUIDE.md revisado
- [ ] TP8-RESUMEN.md leído
- [ ] Screenshots de evidencia guardados
- [ ] Documento PDF/Markdown para entregar

### Defensa:
- [ ] Leídas las 29 preguntas del TP
- [ ] Practicadas respuestas en voz alta
- [ ] Demo preparada (saber mostrar cambios en vivo)
- [ ] URLs memorizadas o anotadas

---

## ⏱️ Tiempo Estimado Total

| Actividad | Tiempo |
|-----------|--------|
| Paso 1: Revisar implementación | 30 min |
| Paso 2: Configurar GitHub | 10 min |
| Paso 3: Crear cuenta Render | 5 min |
| Paso 4: Crear servicios Render | 20 min |
| Paso 5: Configurar secrets | 10 min |
| Paso 6: Primer deploy | 15 min |
| Paso 7: Publicar imágenes | 5 min |
| Paso 8: Tomar evidencia | 30 min |
| Paso 9: Completar docs | 1 hora |
| Paso 10: Preparar defensa | 2 horas |
| **TOTAL** | **~5 horas** |

---

## 🆘 Si algo falla...

### Pipeline falla en "Build and Test"
- Revisar logs en GitHub Actions
- Probablemente un test falló
- Correr `npm test` localmente para debuggear

### Pipeline falla en "Build and Push Images"
- Verificar permisos de GitHub Actions (Paso 2A)
- Verificar que `GITHUB_TOKEN` tenga permiso de escritura

### Deploy falla en Render
- Verificar que las imágenes de GHCR sean públicas (Paso 7)
- Revisar logs en Render dashboard
- Verificar que el Image URL esté correcto

### Health check falla
- Esperar 60 segundos (cold start en free tier)
- Verificar que el puerto sea correcto (3000 backend, 80 frontend)
- Revisar logs en Render

### No puedo aprobar deploy a PROD
- Verificar que creaste el Environment "Production" (Paso 2B)
- Verificar que te agregaste como reviewer

---

## ✅ Cuando todo esté listo...

**¡Felicitaciones!** 🎉

Has implementado una solución profesional de contenedores con CI/CD, completamente gratis, que cumple todas las consignas del TP8.

**Próximo paso**: Defender oralmente tu trabajo explicando cada decisión técnica que tomaste.

**Recuerda**: Lo importante no es qué servicios usaste, sino que entiendas **por qué** los elegiste y **cómo** funcionan.

---

**¡Éxitos con la defensa!** 🚀
