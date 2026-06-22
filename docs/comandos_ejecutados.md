# 🛠️ Comandos Ejecutados durante el Proceso

Este documento resume los comandos técnicos utilizados para la preparación, análisis, remediación y despliegue del reto técnico DevSecOps.

## 1. Preparación del Entorno y Git Flow

```powershell
# Clonado del repositorio base
git clone https://github.com/aelassas/microservices.git devsecops-challenge

# Inicialización de ramas según metodología Git Flow
git checkout -b develop
git checkout -b feature/hardening-catalog

# Instalación de herramientas de seguridad (Trivy) via Winget
winget install Aquasecurity.Trivy
```

## 2. Escaneo Inicial (Línea Base)

```powershell
# Construcción de la imagen vulnerable original
docker build -t catalog-microservice:vulnerable -f src/microservices/CatalogMicroservice/Dockerfile .

# Análisis de vulnerabilidades en la imagen (OS y Librerías)
trivy image --severity CRITICAL,HIGH,MEDIUM --format table -o trivy-report-before.txt catalog-microservice:vulnerable

# Análisis de configuración del Dockerfile (Misconfigurations)
trivy config src/microservices/CatalogMicroservice/Dockerfile

# Análisis del código fuente (Filesystem / SCA)
trivy fs --scanners vuln,secret,misconfig --format table -o trivy-report-fs.txt src/microservices/CatalogMicroservice/
```

## 3. Remediación y Construcción Hardened

```powershell
# Construcción de la nueva imagen optimizada y segura
docker build -t catalog-microservice:hardened -f src/microservices/CatalogMicroservice/Dockerfile.hardened .

# Pruebas de ejecución local (Opcional)
docker run --rm -p 8080:8080 catalog-microservice:hardened
```

## 4. Verificación Final de Seguridad

```powershell
# Re-escaneo de la imagen hardened para asegurar 0 vulnerabilidades
trivy image --severity CRITICAL,HIGH,MEDIUM --format table -o trivy-report-after.txt catalog-microservice:hardened

# Generación de SBOM (Software Bill of Materials) - Valor agregado
trivy image --format cyclonedx -o sbom.json catalog-microservice:hardened
```

## 5. Gestión de Repositorio y Entrega

```powershell
# Sincronización de ramas locales
git add .
git commit -m "feat: hardening microservice and security documentation"

# Configuración de remote y push (Sustituir por tu URL si es necesario)
git remote add origin https://github.com/elkingutierrex/devsecops-challenge.git
git branch -M main
git push -u origin main
git push origin develop
git push origin feature/hardening-catalog
```

---
**Nota:** Estos comandos fueron ejecutados en una terminal de PowerShell sobre un entorno Windows con Docker Desktop habilitado.
