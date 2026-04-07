# Laboratorio de GitHub Actions y Automatización

Este repositorio hace parte de la capacitación práctica de **GitHub Actions, CI/CD y automatización**.

El objetivo de estos laboratorios es que los participantes practiquen la creación y uso de pipelines, incluyendo:

* Creación de workflows
* Ejecución de pipelines (CI)
* Uso de matrix, jobs y runners
* Manejo de artefactos y cache
* Workflows reutilizables
* Manejo de errores y seguridad
* Publicación de paquetes en GitHub Packages
* Versionamiento y releases con semver
* Automatización de releases

## Reglas del laboratorio

1. No hacer **push directo a la rama `main`**.
2. Cada participante trabajará en su propia rama (`feature/tu-nombre`).
3. Todo cambio debe realizarse mediante **Pull Request**.
4. Cada Pull Request debe tener **al menos un revisor**.
5. No aprobar tu propio Pull Request.
6. Validar que los pipelines se ejecuten correctamente antes de hacer merge.

## Estructura del repositorio

* `src/` → Código fuente del proyecto
* `tests/` → Pruebas automatizadas
* `.github.example/` → Ejemplos de workflows
* `.github/workflows/` → Workflows reales (los que crearán los integrantes)
* `docs/` → Guía de laboratorios

## Flujo de trabajo

1. Crear o usar tu rama personal
2. Realizar cambios en el laboratorio correspondiente
3. Ejecutar pruebas localmente (`npm test`)
4. Hacer commit y push
5. Crear Pull Request
6. Validar ejecución de pipelines en **Actions**
7. Revisar y aprobar cambios
8. Hacer merge a `main` (cuando aplique)
