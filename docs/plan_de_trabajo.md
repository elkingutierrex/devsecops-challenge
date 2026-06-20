# 🛡️ Plan de Trabajo: Hardening de Imagen Docker para Microservicio .NET (ASP.NET Core)

## Introducción
Este documento detalla el paso a paso de las acciones que realizaremos para cumplir el reto técnico de **DevSecOps** sobre el repositorio de ejemplo `aelassas/microservices`. El objetivo último es remediar las vulnerabilidades de la imagen original y mejorar la seguridad del código.

---

## Fases de Ejecución

### Fase 1: Descubrimiento y Preparación del Entorno
- [x] **Clonar el proyecto:** Descargamos el repositorio `aelassas/microservices` dentro de tu ruta local `d:\projects\personalProjects\devsecops-challenge`.
- [ ] **Configurar Git Flow:** Inicializar el repositorio con la estructura de ramas clásica (p.ej., `main`, `develop` y crearemos una rama `feature/catalog-hardening`).
- [ ] **Validación de herramientas:** Comprobaremos que Docker y Trivy se encuentran instalados en el entorno (Windows).
- [ ] **Construcción Base:** Compilaremos localmente la imagen del microservicio **CatalogMicroservice** bajo su etiqueta `vulnerable`.

### Fase 2: Escaneo inicial (línea base)
- [ ] **Análisis de la imagen:** Ejecutaremos `trivy image` buscando vulnerabilidades tipo `CRITICAL, HIGH, MEDIUM` en las dependencias base de la imagen.
- [ ] **Análisis del Dockerfile (Misconfiguration):** Ejecutaremos `trivy config` para revisar malas prácticas a nivel de configuración de Docker.
- [ ] **Generación de Evidencia:** Guardaremos los logs resultantes en `trivy-report-before.txt` y en JSON, subiéndolos al repositorio como evidencia inicial.

### Fase 3: Escaneo de código fuente y aplicación (SCA)
- [ ] **Trivy Filesystem (FS):** Escaneo de código estático (NuGet o configuraciones) en `src/microservices/CatalogMicroservice`.
- [ ] **Identificación Manual:** Revisión de controladores, configuración en `appsettings.json`, JWT HTTPS Metadata, CORS, etc.
- [ ] **Generación de Evidencia:** Crearemos archivo `trivy-report-fs.txt`.
- [ ] **Aplicación de Correcciones:** Mejoraremos directamente el código C# atacando -por lo menos- dos vulnerabilidades observadas, comiteando los cambios en la rama feature.

### Fase 4: Análisis y Diagnóstico
- [ ] En un documento de diagnóstico adicional e insertado en el **README.md**, tabularemos todas las vulnerabilidades críticas encontradas (CVE), el vector de explotación detectado de forma resumida, y cuál estrategia de mitigación ejecutaremos en la siguiente fase.

### Fase 5: Remediación
- [ ] **Creación o adaptación de Dockerfile:** Construiremos un `Dockerfile.hardened` que incorpore:
  - `.dockerignore` sólido excluyendo `bin/`, `obj/`, y archivos ocultos.
  - Multi-stage build para reducir por completo librerías de builders (SDK).
  - Usuario `app` no privilegiado (UID).
  - Cambio a una imagen base *alpine* o *chiseled* con identificador `sha256:` que represente el último parche libre de vulnerabilidades persistentes.
  - Ocultamiento de tokens en el build y pase de estas como variables de entorno seguras.
- [ ] **Actualización de NuGets:** Eliminar dependencias vulnerables no usadas o subir su versión.

### Fase 6: Verificación final (cero vulnerabilidades)
- [ ] **Construcción Hardened:** Lanzar `docker build` en la versión segura.
- [ ] **Escaneo Hardened:** Analizar la nueva imagen con `trivy` para cerciorar que no haya remanentes CRITICAL/HIGH/MEDIUM. Todo esto en la misma línea de exigencias del reporte-before.
- [ ] **Guardar Evidencia:** Archivo `trivy-report-after.txt`.

### Cierre y Entregables
- [ ] Actualización cabal del `README.md` del devsecops-challenge detallando hallazgos.
- [ ] Integración vía Pull Request hacia `develop` / `main` terminando el flujo de `Git Flow`.
