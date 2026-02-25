# SS Pipeline - Reusable GitHub Actions Workflows

Una colección de workflows reutilizables de GitHub Actions para automatizar procesos CI/CD, análisis de código, construcción de imágenes Docker y gestión de releases.

## 📋 Descripción General

Este repositorio contiene workflows diseñados para ser reutilizados en otros proyectos a través de `workflow_call`. Incluye soluciones para:

- ✅ Integración continua (CI) para Go y Node.js
- 🐳 Construcción de imágenes Docker
- 📊 Análisis de lenguajes de código
- 🏷️ Validación de etiquetas en PRs
- 📝 Generación automática de CHANGELOG
- 🚀 Gestión automatizada de releases

## 🔧 Workflows Disponibles

### 1. **CI Go** (`ci-go.yml`)

Workflow reusable para proyectos Go que ejecuta tests, análisis estático y construcción.

#### Inputs
- `go-version` (string, optional): Versión de Go a usar. Default: `"1.21"`

#### Pasos
- Configuración de Go con caché
- Descarga de dependencias (`go mod download`)
- Análisis estático (`go vet`)
- Ejecución de tests con cobertura (`go test`)
- Construcción del proyecto (`go build`)

#### Ejemplo de uso

```yaml
name: My Go Project CI
on:
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: slice-soft/ss-pipeline/.github/workflows/ci-go.yml@main
    with:
      go-version: "1.21"
```

---

### 2. **CI Node.js** (`ci-node.yml`)

Workflow reusable para librerías y proyectos Node.js.

#### Inputs
- `node-version` (string, optional): Versión de Node.js a usar. Default: `"22"`

#### Pasos
- Configuración de Node.js
- Caché inteligente de `node_modules`
- Instalación de dependencias
- Ejecución de tests
- Linting (si está configurado)
- Construcción (si está configurada)

#### Ejemplo de uso

```yaml
name: My Node.js Library CI
on:
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: slice-soft/ss-pipeline/.github/workflows/ci-node.yml@main
    with:
      node-version: "22"
```

---

### 3. **Validate PR** (`validate-pr.yml`)

Workflow reusable que valida que los PRs tengan etiquetas semánticas de versionado (patch, minor, major).

#### Requisitos
- Al menos una de las siguientes etiquetas debe estar presente: `patch`, `minor`, `major`

#### Pasos
- Verifica la presencia de etiquetas semver
- Falla si ninguna etiqueta está presente
- Proporciona un mensaje claro del error

#### Ejemplo de uso

```yaml
name: Validate PR
on:
  pull_request:
    types: [opened, labeled, unlabeled, synchronize]
    branches: [main]

jobs:
  validate:
    uses: slice-soft/ss-pipeline/.github/workflows/validate-pr.yml@main
```

---

### 4. **Analyze Code** (`analyze-code.yml`)

Workflow reusable que utiliza GitHub Linguist para analizar los lenguajes presentes en el repositorio.

#### Inputs
- `workdir` (string, optional): Directorio de trabajo. Default: `"."`

#### Pasos
- Configuración de Ruby
- Instalación de GitHub Linguist gem
- Análisis de lenguajes del repositorio
- Generación de reporte
- Subida de artefacto

#### Ejemplo de uso

```yaml
name: Analyze Repository
on:
  push:
    branches: [main]

jobs:
  analyze:
    uses: slice-soft/ss-pipeline/.github/workflows/analyze-code.yml@main
    with:
      workdir: "."
```

---

### 5. **Create Release** (`create-release.yml`)

Workflow reusable que genera automáticamente un CHANGELOG, crea un tag de versión y publica un release en GitHub.

#### Características
- Genera CHANGELOG según [Conventional Commits](https://www.conventionalcommits.org/)
- Crea automáticamente tags de versión semántica (semver)
- Crea un release en GitHub con el CHANGELOG
- Actualiza el tag major para facilitar referencias

#### Pasos
- Genera CHANGELOG y versión usando Conventional Commits
- Extrae la versión major del tag
- Crea un release en GitHub
- Actualiza el tag de versión major

#### Ejemplo de uso

```yaml
name: Release Pipeline
on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  release:
    uses: slice-soft/ss-pipeline/.github/workflows/create-release.yml@main
```

---

### 6. **Build Docker** (`build-docker.yml`)

Workflow reusable para construir y publicar imágenes Docker en GitHub Container Registry.

#### Inputs
- `workdir` (string, required): Directorio de trabajo
- `dockerfile` (string, required): Ruta del Dockerfile
- `image_name` (string, required): Nombre de la imagen Docker
- `version` (string, required): Tag de versión para la imagen

#### Secrets
- `SSH_PRIVATE_KEY` (required): Clave privada SSH para acceso a repositorios privados

#### Características
- Análisis de código con Linguist
- Construcción con Docker BuildKit habilitado
- Push a GitHub Container Registry
- Firma de imágenes con Cosign

#### Ejemplo de uso

```yaml
name: Build Docker Image
on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    uses: slice-soft/ss-pipeline/.github/workflows/build-docker.yml@main
    with:
      workdir: "."
      dockerfile: "Dockerfile"
      image_name: "my-app"
      version: "1.0.0"
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

---

## 🚀 Uso en Tus Repositorios

### Paso 1: Agregar un workflow que use los workflows reusables

Crea un archivo en `.github/workflows/` en tu repositorio:

```yaml
name: CI Pipeline
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  validate:
    uses: slice-soft/ss-pipeline/.github/workflows/validate-pr.yml@main
    
  test-go:
    uses: slice-soft/ss-pipeline/.github/workflows/ci-go.yml@main
    with:
      go-version: "1.21"
```

### Paso 2: Configurar permisos

Asegúrate de que tu repositorio tiene los permisos necesarios en `Settings > Actions > General`:

- ✅ Actions permissions: "Allow all actions and reusable workflows"
- ✅ Workflow permissions: "Read and write permissions"

### Paso 3: Configurar secretos (si es necesario)

Para workflows que requieren secretos (como `build-docker.yml`), agrega los secretos en:

`Settings > Secrets and variables > Actions`

---

## 📋 Requisitos Previos

### Para CI Go
- El proyecto debe tener un archivo `go.mod`
- Tests en formato estándar de Go

### Para CI Node.js
- El proyecto debe tener `package.json`
- `package-lock.json` para reproducibilidad

### Para Create Release
- Commits que sigan [Conventional Commits](https://www.conventionalcommits.org/)
- Permisos de escritura en el repositorio

### Para Build Docker
- Dockerfile presente en la ruta especificada
- GitHub Container Registry configurado

---

## 🔐 Secretos Requeridos

### `SSH_PRIVATE_KEY` (para build-docker.yml)

Se usa para acceso a repositorios privados durante la construcción:

```bash
# Generar una clave SSH (si no tienes ya una)
ssh-keygen -t ed25519 -f gh-action-key -N ""

# Agregar a SSH_PRIVATE_KEY secret en GitHub
cat gh-action-key | base64
```

---

## 📝 Convenciones de Commits

Para que el workflow `create-release.yml` funcione correctamente, sigue las convenciones de commits:

```
feat: Nueva característica (MINOR)
fix: Corrección de bug (PATCH)
perf: Mejora de rendimiento (PATCH)
docs: Cambios de documentación
refactor: Refactorización sin cambio de funcionalidad
test: Agregación o modificación de tests
chore: Cambios en herramientas, configuración, etc.
```

**Ejemplo:**
```
feat(auth): agregar autenticación de dos factores

BREAKING CHANGE: La API de login ha cambiado
```

---

## 🏷️ Etiquetas Semánticas

El workflow `validate-pr.yml` requiere una de las siguientes etiquetas en cada PR:

| Etiqueta | Significado | Versión |
|----------|-------------|---------|
| `patch` | Correcciones de bugs | 1.0.x |
| `minor` | Nuevas características | 1.x.0 |
| `major` | Cambios incompatibles | x.0.0 |

---

## 🐛 Troubleshooting

### El workflow de release no genera CHANGELOG

- Verifica que los commits sigan [Conventional Commits](https://www.conventionalcommits.org/)
- Asegúrate de que no hay tags previos en el repositorio

### El build Docker falla con permisos SSH

- Verifica que la clave SSH está correctamente codificada en base64
- Asegúrate de que el host está agregado a `known_hosts`

### El CI no usa el caché de dependencias

- Verifica que `package-lock.json` (Node.js) o `go.sum` (Go) estén versionados
- El hash del archivo debe ser determinístico

---

## 📚 Recursos Adicionales

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusable Workflows](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions#jobsjob_iduses)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

---

## 📄 Licencia

Este proyecto es de código abierto y está disponible para uso en otros proyectos.

---

## 🤝 Contribuciones

Para contribuir mejoras a estos workflows, por favor:

1. Crea un branch con tu cambio
2. Asegúrate de etiquetar tu PR correctamente
3. Los cambios serán validados antes de ser mergeados

---

**Última actualización:** 25 de febrero de 2026
