# SS-Pipeline

Este repositorio contiene un pipeline de integración y despliegue continuo (CI/CD) diseñado para automatizar tareas clave en el desarrollo de software. 

## Propósito

El objetivo principal de este pipeline es simplificar y estandarizar el proceso de construcción, análisis y despliegue de aplicaciones. Esto incluye:

- **Construcción de imágenes Docker**: Automatiza la creación de imágenes Docker utilizando configuraciones específicas.
- **Análisis de código**: Realiza un análisis del repositorio utilizando GitHub Linguist para identificar los lenguajes de programación utilizados.
- **Creación de releases**: Genera versiones semánticas (SemVer) y publica releases en GitHub automáticamente.
- **Ejecutar Tareas Previas**: Permite ejecutar tareas específicas antes de la construcción y despliegue, como análisis de código o pruebas.
- **Pipeline de Release**: Automatiza el proceso de creación y publicación de releases en GitHub.
- **Pipeline de Pull Requests**: Facilita la revisión y validación de cambios antes de ser fusionados en la rama principal.

## Características principales

1. **Pipeline de construcción y despliegue**:
   - Construcción de imágenes Docker con soporte para etiquetas de versión y `latest`.
   - Publicación de imágenes en el GitHub Container Registry (ghcr.io).

2. **Análisis de código**:
   - Utiliza GitHub Linguist para analizar los lenguajes del repositorio y generar un reporte.

3. **Gestión de releases**:
   - Genera etiquetas de versión semántica automáticamente.
   - Publica releases en GitHub con información relevante.

## Cómo funciona

El pipeline se activa en diferentes escenarios:

- **Pull Requests cerrados en la rama `main`**: Desencadena el pipeline de release.
- **Llamadas a workflows específicos**: Permite ejecutar tareas como análisis de código o construcción de imágenes Docker mediante `workflow_call`.

## Configuración

El pipeline utiliza GitHub Actions y requiere ciertos secretos configurados en el repositorio:

- `SSH_PRIVATE_KEY`: Clave privada para la construcción de imágenes Docker.
- `GITHUB_TOKEN`: Token de GitHub para autenticación y publicación de releases.

## Requisitos

- Docker instalado en el entorno de ejecución.
- Configuración de GitHub Actions en el repositorio.

Este pipeline está diseñado para ser modular y reutilizable, facilitando su integración en diferentes proyectos.