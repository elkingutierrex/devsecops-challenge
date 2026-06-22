# 🔍 Diagnóstico Detallado de Seguridad - CatalogMicroservice

Este documento desglosa cada vulnerabilidad relevante encontrada durante el escaneo inicial y la lógica detrás de su remediación.

## 1. Análisis de Vulnerabilidades (CVE)

### CVE-2024-21319
- **Paquete afectado:** `System.Text.Json` v8.0.0
- **Severidad:** CRITICAL
- **Componente:** NuGet Dependency (.NET Runtime)
- **Vector de explotación:** Un atacante puede enviar un JSON malformado que cause una recursión infinita o un consumo excesivo de memoria, provocando una Denegación de Servicio (DoS) en el servidor ASP.NET Core.
- **Estrategia de remediación:** Actualización de la dependencia a la versión 8.0.4 o superior. En el hardening, se forzó el uso de la imagen base de .NET 8 con los últimos parches de seguridad.

### CVE-2023-5363
- **Paquete afectado:** `libssl3` v3.0.11
- **Severidad:** HIGH
- **Componente:** OS Package (Debian 12)
- **Vector de explotación:** Buffer overflow en el manejo de certificados TLS. Podría permitir la ejecución remota de código (RCE) o la caída del servicio de cifrado.
- **Estrategia de remediación:** Migración a **Alpine Linux 3.20**. Alpine utiliza `musl` y versiones más recientes y compactas de `openssl`/`libssl`, resolviendo los CVEs presentes en la imagen base estándar de Debian.

### CVE-2023-45853
- **Paquete afectado:** `zlib1g`
- **Severidad:** HIGH
- **Componente:** OS Package (Debian 12)
- **Vector de explotación:** Error de validación en MiniZ que permite desbordamiento de búfer durante la descompresión de archivos maliciosos.
- **Estrategia de remediación:** Eliminación del paquete mediante el uso de una imagen base "Alpine", la cual no incluye estas librerías vulnerables por defecto o utiliza versiones parcheadas.

---

## 2. Análisis de Configuraciones (Misconfigurations)

### DS002 - Running as Root
- **Riesgo:** Si un atacante logra explotar una vulnerabilidad en la aplicación y el contenedor corre como `root`, el atacante obtiene privilegios totales sobre el sistema de archivos del contenedor y facilitaría un "container escape".
- **Remediación:** Se configuró explícitamente `USER app` en el Dockerfile. El microservicio ahora corre con privilegios mínimos.

### Hardcoded JWT Secret
- **Riesgo:** El secreto de firma de tokens está expuesto en el repositorio. Cualquier persona con acceso al código (o que logre leer el sistema de archivos) puede generar tokens válidos (admin) y suplantar a cualquier usuario.
- **Remediación:** Se eliminó el secreto de `appsettings.json`. Se instruye al usuario a suministrarlo mediante la variable de entorno `JWT__SECRET`.

---

## 3. Seguridad a Nivel de Aplicación

| Hallazgo | Impacto | Mitigación |
|----------|---------|------------|
| `RequireHttpsMetadata = false` | Permite ataques Man-in-the-Middle (MitM) al interceptar tokens en canales no cifrados. | Se cambió a `true` en el pipeline de autenticación. |
| Falta de validación en POST | Permite la persistencia de datos corruptos o ataques de inyección básica. | Se agregó validación (`BadRequest`) en `CatalogController`. |
| CORS Permisivo (`*`) | Permite que cualquier sitio web realice peticiones al API (si no hay protecciones adicionales). | Se recomienda configurar una política de CORS estricta en producción (actualmente en evaluación). |

---
**Documento generado para el reto técnico de DevSecOps.**
