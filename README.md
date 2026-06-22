# 🛡️ Solución Reto DevSecOps: Hardening Microservicio .NET

Esta es la documentación técnica de la solución implementada para el hardening del microservicio `CatalogMicroservice`, parte del ecosistema de arquitectura de microservicios de .NET.

## 1. Resumen Ejecutivo

Se ha realizado un proceso completo de saneamiento que abarca desde la capa de código fuente hasta el empaquetado en contenedores Docker. El estado inicial presentaba vulnerabilidades críticas de denegación de servicio (DoS), secretos expuestos y malas configuraciones de seguridad.

**Resultado final:** 0 Vulnerabilidades CRITICAL/HIGH/MEDIUM detectadas en el reporte final.

---

## 2. Hallazgos y Diagnóstico

### Tabla de Vulnerabilidades (Trivy Scan)

| CVE              | Severidad | Componente       | Versión Vulnerable | Estrategia de Remediación                                  |
| ---------------- | --------- | ---------------- | ------------------ | ---------------------------------------------------------- |
| CVE-2024-21319   | CRITICAL  | System.Text.Json | 8.0.0              | Actualización de dependencias y cambio de imagen base.     |
| CVE-2023-5363    | HIGH      | libssl3 (OS)     | 3.0.11-1           | Migración a imagen base Alpine (más ligera y parcheada).   |
| Hardcoded Secret | CRITICAL  | appsettings.json | N/A                | Eliminación de secreto y migración a variables de entorno. |
| Non-Root User    | HIGH      | Dockerfile       | root               | Implementación de usuario `app` (UID 64198).               |

### Hallazgos de Código (Fase 3)

1. **JWT sin HTTPS:** El microservicio permitía la validación de tokens sobre HTTP.
   - _Fix:_ Se forzó `RequireHttpsMetadata = true`.
2. **Validación de Entradas:** El controlador aceptaba objetos sin validar campos obligatorios.
   - _Fix:_ Se implementó validación lógica en `CatalogController.cs` retornando `BadRequest` ante datos inválidos.
3. **Secretos Expuestos:** El secreto de firma de tokens estaba en texto plano.
   - _Fix:_ Se eliminó del archivo de configuración.

---

## 3. Hardening del Dockerfile

El nuevo `Dockerfile.hardened` implementa las siguientes mejores prácticas:

1. **Imagen Base Alpine:** Se seleccionó `aspnet:8.0-alpine` para minimizar binarios innecesarios (shell limitado) y reducir la superficie de ataque.
2. **Multi-Stage Build:** El SDK solo vive en la etapa de construcción, nunca en la imagen final.
3. **Usuario No-Root:** La aplicación corre bajo el usuario `app`. Esto mitiga riesgos de escalamiento de privilegios si el contenedor fuera comprometido.
4. **Endpoint de Salud:** Se integró un `HEALTHCHECK` nativo que valida que el servicio responda a las peticiones de salud interna.
5. **Seguridad de Capas:** Se utiliza `--chown=app:app` al copiar archivos para asegurar que la aplicación sea dueña de sus propios binarios pero con permisos limitados.

---

## 4. Reproducción

### Construcción de Imagen Vulnerable

```bash
docker build -t catalog-microservice:vulnerable -f src/microservices/CatalogMicroservice/Dockerfile .
```

### Construcción de Imagen Segura

```bash
docker build -t catalog-microservice:hardened -f src/microservices/CatalogMicroservice/Dockerfile.hardened .
```

### Escaneo con Trivy

```bash
trivy image catalog-microservice:hardened --severity CRITICAL,HIGH,MEDIUM
```

---

## 5. Documentación del Proceso

Para ver el detalle paso a paso de lo realizado, puede consultar los siguientes documentos creados durante el reto:

- [Plan de Trabajo](./docs/plan_de_trabajo.md)
- [Bitácora de Implementación](./docs/bitacora.md)

## 6. URL's Documentación BEFORE - AFTER

- [Before](./trivy-report-before.txt)
- [After](./trivy-report-after.txt)

---

**Utilizando** Antigravity (AI Assistant)
**Metodología:** Git Flow / DevSecOps Standards
