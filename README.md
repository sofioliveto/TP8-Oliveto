# TP8: Contenedores y Automatización

## 📋 Descripción

Trabajo Práctico 8 de Ingeniería de Software III - Universidad Católica de Córdoba

Implementación completa de **contenedores Docker** con **CI/CD automático** usando servicios **100% gratuitos**.

## 🏗️ Stack Tecnológico

- **Aplicación**: Node.js + Express + SQLite (backend) + HTML/CSS/JS (frontend)
- **Contenedores**: Docker (multi-stage builds)
- **Container Registry**: GitHub Container Registry (ghcr.io)
- **Hosting**: Render.com (free tier)
- **CI/CD**: GitHub Actions
- **Testing**: Jest (unit) + Cypress (E2E)

## 🚀 Inicio Rápido

### Desarrollo Local

```bash
# Clonar repositorio
git clone https://github.com/sofioliveto/TP8-Oliveto.git
cd TP8-Oliveto

# Backend
cd backend
npm install
npm start

# Frontend (en otra terminal)
cd frontend
npm install
# Abrir index.html en browser
```

### Con Docker Local

```bash
# Build backend image
docker build -t palabras-backend:local -f docker/backend/Dockerfile ./backend
docker run -p 3000:3000 palabras-backend:local

# Build frontend image
docker build -t palabras-frontend:local -f docker/frontend/Dockerfile ./frontend
docker run -p 8080:80 palabras-frontend:local
```

## 📚 Documentación

### Para Estudiantes
1. **[CHECKLIST-TAREAS.md](./CHECKLIST-TAREAS.md)** - ¡EMPIEZA AQUÍ! Lista paso a paso de qué hacer
2. **[TP8-RESUMEN.md](./TP8-RESUMEN.md)** - Resumen ejecutivo de la implementación
3. **[TP8-SETUP-GUIDE.md](./TP8-SETUP-GUIDE.md)** - Guía de configuración detallada
4. **[TP8-DOCUMENTACION-TECNICA.md](./TP8-DOCUMENTACION-TECNICA.md)** - Documentación completa con justificaciones

### Para Profesores
- Revisar `TP8-DOCUMENTACION-TECNICA.md` para ver decisiones arquitectónicas
- Verificar pipeline en GitHub Actions: [![CI/CD](https://github.com/sofioliveto/TP8-Oliveto/actions/workflows/ci-cd-containers.yml/badge.svg)](https://github.com/sofioliveto/TP8-Oliveto/actions/workflows/ci-cd-containers.yml)

## ✅ Consignas Completadas

- [x] Container Registry configurado (GHCR)
- [x] Dockerfiles optimizados (multi-stage)
- [x] Pipeline CI/CD completo (GitHub Actions)
- [x] Deploy automático a QA
- [x] Deploy manual a PROD (con approval)
- [x] Versionado semántico de imágenes
- [x] Quality gates implementados
- [x] Documentación exhaustiva

## 🌐 URLs de los Ambientes

- **Backend QA**: https://palabras-backend-qa.onrender.com
- **Frontend QA**: https://palabras-frontend-qa.onrender.com
- **Backend PROD**: https://palabras-backend-prod.onrender.com
- **Frontend PROD**: https://palabras-frontend-prod.onrender.com

## 🧪 Testing

```bash
# Unit tests backend
cd backend
npm test

# Unit tests frontend
cd frontend
npm test

# E2E tests (requiere backend corriendo)
npx cypress run --config baseUrl=http://localhost:3000
```

## 💰 Costos

**Total: $0/mes** (100% gratuito)

- Container Registry (GHCR): Gratis
- CI/CD (GitHub Actions): Gratis (2000+ min/mes)
- Hosting QA (Render): Gratis (750 hrs/mes)
- Hosting PROD (Render): Gratis (750 hrs/mes)

## 📦 Estructura del Proyecto

```
TP8-Oliveto/
├── .github/
│   └── workflows/
│       └── ci-cd-containers.yml    # Pipeline CI/CD
├── docker/
│   ├── backend/
│   │   └── Dockerfile              # Backend multi-stage
│   └── frontend/
│       └── Dockerfile              # Frontend con Nginx
├── backend/                        # API Node.js + Express
├── frontend/                       # UI HTML/CSS/JS
├── cypress/                        # Tests E2E
├── TP8-DOCUMENTACION-TECNICA.md   # Documentación completa
├── TP8-SETUP-GUIDE.md             # Guía de setup
├── TP8-RESUMEN.md                 # Resumen ejecutivo
└── CHECKLIST-TAREAS.md            # Lista de tareas pendientes
```

## 🛠️ Configuración Requerida

Ver [CHECKLIST-TAREAS.md](./CHECKLIST-TAREAS.md) para pasos detallados.

Resumen:
1. Crear servicios en Render.com (4 servicios)
2. Configurar GitHub Secrets (8 secrets)
3. Configurar GitHub Environments (QA + Production)
4. Mergear PR para ejecutar pipeline

## 🎓 Trabajo Práctico

Este repositorio es la solución del TP8 de Ingeniería de Software III.

**Consigna**: Implementar contenedores y CI/CD usando servicios cloud, con documentación de decisiones técnicas.

**Entregables**:
- ✅ Código funcionando
- ✅ Pipeline CI/CD operativo
- ✅ Apps deployadas en QA y PROD
- ✅ Documentación técnica justificando decisiones
- ⏸️ Defensa oral (pendiente)

## 📞 Contacto

**Estudiante**: Sofía Oliveto  
**Materia**: Ingeniería de Software III  
**Universidad**: Universidad Católica de Córdoba

---

**Nota**: Este TP fue implementado completamente con servicios gratuitos. No se requiere presupuesto para reproducirlo.
