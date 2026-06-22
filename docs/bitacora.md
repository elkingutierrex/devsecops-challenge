# 📓 Bitácora de Trabajo - Reto DevSecOps

Esta bitácora registra de manera cronológica todas las acciones, análisis y decisiones tomadas durante el proceso de hardening del microservicio `CatalogMicroservice`.

---

## [2026-06-20] - Sesión de Tarde

### 1. Preparación del Entorno
- **Acción:** Clonado del repositorio sugerido `https://github.com/aelassas/microservices` en el directorio local `devsecops-challenge`.
- **Acción:** Creación de `docs/plan_de_trabajo.md` para definir el alcance y las fases.
- **Acción:** Inicialización de Git Flow. Ramas creadas: `develop` y `feature/hardening-catalog`.
- **Acción:** Instalación de **Trivy** mediante `winget` en el entorno Windows para realizar los escaneos de seguridad.

### 2. Análisis de Vulnerabilidades (Línea Base)
- **Análisis de Imagen:** Se identificó que la imagen original usa Debian 12 como base. Aunque es estable, contiene paquetes con CVEs conocidos (libssl3, zlib1g).
- **Análisis de Filesystem (SCA):**
    - Se detectó un secreto JWT hardcodeado en `appsettings.json`: `9095a623-a23a-481a-aa0c-e0ad96edc103`.
    - La configuración de JWT en `Middleware/Extentions.cs` tenía `RequireHttpsMetadata = false`, lo cual permite tráfico no cifrado para la autenticación.
    - El microservicio no realizaba validaciones de entrada en el `POST` de `CatalogController`, permitiendo la creación de items incompletos o malformados.
- **Evidencias guardadas:** Se generaron `trivy-report-before.txt` y `trivy-report-fs.txt`.

### 3. Remediación del Código de Aplicación
- **Cambio 1 (Seguridad JWT):** Se actualizó `Extentions.cs` para forzar `RequireHttpsMetadata = true`. Esto obliga a que el microservicio solo valide tokens a través de canales HTTPS.
- **Cambio 2 (Sanitización/Validación):** Se modificó `CatalogController.cs` para incluir una validación explícita. Si el `CatalogItem` es nulo o el nombre está vacío, se retorna un `BadRequest(400)`.
- **Cambio 3 (Gestión de Secretos):** Se eliminó el valor del secreto de `appsettings.json` y se dejó un marcador de posición, indicando que debe ser provisto por variables de entorno o un Secret Manager.

### 4. Hardening de la Imagen Docker
- **Acción:** Creación de `Dockerfile.hardened`.
- **Decisión Técnica (Base Image):** Se cambió de Debian a **Alpine Linux (3.20)**. 
    - *Razón:* Reducción drástica de la superficie de ataque y del tamaño de la imagen (de ~200MB a ~100MB).
- **Decisión Técnica (Usuario):** Se activó el usuario `app` (UID 64198) incluido en las imágenes de .NET 8. Se dejó de usar `root`.
- **Decisión Técnica (Puerto):** Al ser un usuario no-root, se cambió el puerto de escucha del 80 al **8080**.
- **Decisión Técnica (Multi-stage):** Se separó claramente el SDK de la imagen de runtime para asegurar que no queden herramientas de compilación en producción.
- **Decisión Técnica (Salud):** Se implementó un `HEALTHCHECK` usando `wget` hacia `/healthz`.

### 5. Verificación Final
- **Evidencia:** Se generó `trivy-report-after.txt` mostrando 0 vulnerabilidades de severidad CRITICAL, HIGH o MEDIUM después de aplicar el hardening.

### 6. Entrega y Sincronización (Final)
- **Acción:** Configuración del repositorio remoto `https://github.com/elkingutierrex/devsecops-challenge.git`.
- **Acción:** Sincronización de ramas locales (`main`, `develop`, `feature/hardening-catalog`) con el repositorio remoto del usuario.
- **Acción:** Creación de Workflow automatizado en `.github/workflows/security-scan.yml` para asegurar la continuidad del hardening en el pipeline de CI/CD.
- **Acción:** Actualización exhaustiva del `README.md` con tablas de vulnerabilidades y estrategias de mitigación.

---
*Estado: Reto finalizado y documentado.*
