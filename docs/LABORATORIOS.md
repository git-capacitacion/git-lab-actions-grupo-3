# Laboratorios prácticos

## Laboratorio 1 - Configuración inicial del proyecto

1. Clonar el repositorio.
2. Crear una rama con tu nombre. **Nombre sugerido:** `feature/tu-nombre`.
3. Trabajar SIEMPRE sobre esta misma rama durante todos los laboratorios.
4. Instalar jest para ejecutar los tests:

    ```powershell
    npm install jest --save-dev
    ```

5. Ejecutar los tests localmente:

    ```powershell
    npm test
    ```

6. Abrir el archivo `package.json`.
7. Completar:
    - `name`: debe tener scope (ejemplo: `@nombre-organizacion/nombre-repositorio`)
    - `description`
    - `author`: tu nombre
8. Completar `repository` cambiando la url con el nombre de la organización y el repositorio:

    ```json
    {
      "type":"git",
      "url":"https://github.com/nombre-organizacion/nombre-repositorio.git"
    }
    ```

9. Completar `publishConfig`:

    ```json
    {
      "registry":"https://npm.pkg.github.com"
    }
    ```

10. Crear el archivo `.npmrc` en la raíz del proyecto y agregar el mismo contenido de `.npmrc.example`.
11. Ajustar el scope en `.npmrc` con el nombre de la organización y del repositorio (`@nombre-organizacion/nombre-repositorio`).
12. Guardar cambios.
13. Subir tu rama al repositorio.

## Laboratorio 2 - Creación del pipeline (CI)

1. Ir a carpeta `.github/workflows/`.
2. Crear el archivo `ci.yml`.
3. Ir a la carpeta `.github.example/workflows/`.
4. Copiar el contenido de `ci.example.yml` y pegarlo en `ci.yml`.
5. Completar:
    - `name`: colocarle el nombre `Workflow Laboratorios` al pipeline
    - `on: push`: incluir las ramas `main` y `feature/**`
    - `on: pull_request`: incluir la rama `main`
    - `jobs`: completar el job colocandole el nombre `test`
        - `runs-on:`: colocar como runner la última versión de ubuntu `ubuntu-latest`
6. Agregar pasos (steps) en el job:
    - Checkout repository:

        ```yaml
        uses: actions/checkout@v4
        ```

    - Setup Node:

        ```yaml
        uses: actions/setup-node@v4
        ```

    - Instalar dependencias:

        ```yaml
        run: npm ci
        ```

        Es similar a `npm install` pero `npm ci` se recomienda en CI/CD.

    - Ejecutar tests:

        ```yaml
        run: npm test
        ```

7. Guardar y subir cambios.
8. Validar en **Actions** que el pipeline se ejecuta.

## Laboratorio 3 - Matrix

1. Editar `ci.yml`.
2. Agregar `strategy.matrix` en el job creado en el laboratorio 2.

    ```yaml
    strategy:
      matrix:
        node: [16, 18, 20]
    ```

3. En el paso “**Setup Node**”, agregar:

    ```yaml
    with:
      node-version: ${{ matrix.node }}
    ```

4. Guardar y subir cambios.
5. Ejecutar el pipeline y analizar que sucede.
6. Agregar en `strategy` para limitar los jobs que se ejecutan al tiempo y en caso de que uno falle no cancele el resto de jobs:

    ```yaml
    fail-fast: false
    max-parallel: 2
    ```

7. Guardar y subir cambios.
8. Ejecutar el pipeline y analizar que sucede.

9. Agregar un `include` en `strategy.matrix` para incluir la versión 22 de node.

    ```yaml
    include:
      - node: 22
    ```

10. Agregar un `exclude` en `strategy.matrix` para excluir la versión 16 de node.

    ```yaml
    exclude:
      - node: 16
    ```

11. Guardar y subir cambios.
12. Ejecutar el pipeline y analizar que sucede.

## Laboratorio 4 - Cache y artefactos

1. Editar `ci.yml`.
2. Agregar cache en el paso “**Setup Node**”:

    ```yaml
    with:
      node-version: ${{ matrix.node }}
      cache: 'npm'
    ```

    De esta forma, usamos el caché integrado de `actions/setup-node`.

    Permite automáticamente cachear `~/.npm` y usar como clave el `package-lock.json`.

3. Actualizar el paso llamado “**Run tests**”:

    ```yaml
    run: npm test -- --coverage
    ```

    Esto permite que se genere la carpeta de coverage (métrica que indica qué porcentaje del código está siendo probado), la cual usaremos para guardar los artefactos.

4. Crear un nuevo paso (step) al final del job:

    ```yaml
    - name: Upload coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage-node-${{ matrix.node }}
        path: coverage/
    ```

5. Ejecutar pipeline.
6. Descargar alguno de los artefactos desde Actions y abrir el archivo `index.js` para observar el coverage.

## Laboratorio 5 - Dividir jobs

1. Crear el archivo `ci-updated.yml`.
2. Copiar el contenido de `ci.yml` y pegarlo en el nuevo archivo.
3. Actualizar `name` por “**Workflow Laboratorios - Actualizado**”
4. En `ci-updated.yml`, crear un job arriba del job de “**test**” llamado `build`.
5. En el job “**build**” agregar el `runs-on` y los siguientes pasos (steps):

    ```yaml
    - name: Checkout repository
      uses: actions/checkout@v4
    ```

    ```yaml
    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: 20
        cache: 'npm'
    ```

    ```yaml
    - name: Install dependencies
      run: npm ci
    ```

    Ejecuta el script de `build` que tenemos en el `package.json`, crea la carpeta `dist/` y queda el archivo `math.js` dentro de la carpeta:

    ```yaml
    - name: Build project
      run: npm run build
    ```

    Toma la carpeta `/dist`, la comprime y la guarda como artefacto con el nombre `build`:

    ```yaml
    - name: Upload build
      uses: actions/upload-artifact@v4
      with:
        name: build
        path: dist/
    ```

6. En el job de “**test**” hacer lo siguiente:
    - Eliminar la sección de `strategy`
    - Agregar un `needs` de `build`
    - En los pasos (steps):
        - Actualizar el paso “**Setup Node**” por:

            ```yaml
            - name: Setup Node
              uses: actions/setup-node@v4
              with:
                node-version: 20
            ```

        - Crear un nuevo paso debajo del paso “**Install dependencies**”:

            ```yaml
            - name: Download build
              uses: actions/download-artifact@v4
              with:
                name: build
                path: dist/
            ```

        Esto usa el artefacto creado previamente por el paso de build y lo descomprime en `dist`.

        - Actualizar el paso “Run tests”:

            ```yaml
            - name: Run tests
              run: npm test
            ```

        - Eliminar el “Upload coverage artifact”.

7. Ejecutar pipeline.
8. Descargar el artefacto desde Actions.

## Laboratorio 6 - Seguridad y simulación de errores

1. Crear el workflow `security.yml`.
2. Copiar el contenido de `security.example.yml` y pegarlo en `security.yml`.
3. Completar:
    - `name`: colocarle el nombre `Workflow Security` al pipeline
    - `on: push`: incluir las ramas `main` y `feature/**`
    - `jobs`: completar el job colocandole el nombre `audit`
        - `runs-on:`: colocar como runner la última versión de ubuntu `ubuntu-latest`
4. Agregar pasos (steps) en el job:
    - Checkout repository:

        ```yaml
        uses: actions/checkout@v4
        ```

    - Setup Node:

        ```yaml
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
        ```

    - Instalar dependencias:

        ```yaml
        run: npm ci
        ```

    - Ejecutar comando para buscar vulnerabilidades:

        ```yaml
        - name: Run audit
          run: npm audit --audit-level=high
        ```

5. Ejecutar el pipeline.
6. Simular error instalando una dependencia vulnerable:

    ```powershell
    npm install lodash@4.17.19
    ```

7. Ejecutar pipeline y observar el error.
8. Configurar que el pipeline se siga ejecutando aunque haya un error, en el paso de “**Run audit**” agregar:

    ```yaml
    continue-on-error: true
    ```

9. Ejecutar pipeline y observar que el pipeline se ejecuta correctamente a pesar del error.
10. Arreglar el error actualizando la dependencia:

    ```powershell
    npm install lodash@latest
    ```

11. Ejecutar el pipeline.
12. Agregar un nuevo paso (step) después del paso “**Checkout repository**” para simular una espera de 2 minutos:

    ```yaml
    - name: Simulate long task
      run: sleep 120
    ```

13. Agregar un tiempo de espera de 1 minuto, agregar debajo de `runs-on`:

    ```yaml
    timeout-minutes: 1
    ```

14. Ejecutar pipeline y observar como se cancela.
15. Eliminar la simulación de espera para tener nuevamente funcionando nuestro pipeline (`time-out` y `sleep`).

## Laboratorio 7 - Publicación en GitHub Packages

### Publicar paquete

La primera persona que llegué a este laboratorio, debe tomar el issue `#1` para indicar que se encargará de publicar el paquete.

1. Ir a los ajustes de tu perfil, llegar al final del panel izquierdo y dar click en `Developer Settings`.
2. Dar click en `Personal access tokens` y seleccionar la opción `Tokens (classic)`.
3. Dar click en `Generate new token` y posteriormente en `Generare new token (classic)`.
4. En `Note` coloca el nombre del token y en los `scopes` seleccionar la opción `write:packages`.
5. Generar el token y copiarlo.
6. Hacer PR a `main`.
7. Cambiar a la rama `main` en tu máquina y traer los cambios del repositorio remoto.
8. En tu máquina ejecutar:

    ```powershell
    npm login --registry=https://npm.pkg.github.com
    ```

    - username: tu usuario de GitHub en minúsculas
    - password: token personal

9. Publicar el paquete:

    ```powershell
    export NODE_AUTH_TOKEN=TU_TOKEN
    npm publish
    ```

### Consumir paquete

Esto lo realiza tanto el que subió el paquete como los demás integrantes.

1. Crear una nueva rama llamada `feature/tu-nombre-v2`.
2. Instalar:

    ```powershell
    npm install @nombre-organizacion/nombre-repositorio@1.0.0 \
    --registry=https://npm.pkg.github.com
    ```

3. Crear archivo `index.js` en la raíz del proyecto:

    ```javascript
    const { sum }=require('@nombre-organizacion/nombre-repositorio');
    
    console.log(sum(2,3));
    ```

4. Ejecutar en la consola:

    ```powershell
    node index.js
    ```

5. Validar que funciona, el resultado debe ser "**La suma entre 2 y 3 es: 5**".
6. Guardar cambios.
7. Subir la rama al repositorio remoto.

## Laboratorio 8 - Release manual con tags

1. Cambiar a la rama `feature/tu-nombre`.
2. Crear tag:

    ```bash
    git tag v1.0.0-tu-nombre
    ```

3. Subir tag:

    ```bash
    git push origin v1.0.0-tu-nombre
    ```

4. Ir a GitHub → Releases
5. Crear release basado en ese tag completando:
    - **Release title**: Título del release (`v1.0.0-tu-nombre Pre Release`)
    - **Set as pre-release**: Seleccionar la última opción para marcar nuestro release como un pre release.

## Laboratorio 9 - Automatización de un pre-release

1. En nuestra rama `feature/tu-nombre`, instalar las siguientes dependencias:

    ```powershell
    npm install semantic-release \
      @semantic-release/commit-analyzer \
      @semantic-release/release-notes-generator \
      @semantic-release/npm \
      @semantic-release/github \
      @semantic-release/changelog \
      @semantic-release/git \
      --save-dev
    ```

2. Crear el archivo `.releaserc` en la raíz del proyecto y agregar el siguiente contenido:

    ```json
    {
    "branches": [
      "main",
      {
      "name": "feature/tu-nombre",
      "prerelease": "beta"
      },
      {
      "name": "feature/tu-nombre-v2",
      "prerelease": "alpha"
      }
    ],
    "plugins": [
      "@semantic-release/commit-analyzer",
      "@semantic-release/release-notes-generator",
      "@semantic-release/changelog",
      [
      "@semantic-release/npm",
      {
        "npmPublish": true
      }
      ],
      "@semantic-release/github",
      [
      "@semantic-release/git",
      {
        "assets": ["CHANGELOG.md", "package.json"],
        "message": "chore(release): ${nextRelease.version}"
      }
      ]
    ]
    }
    ```

    `semantic-release` es una librería que usa SemVer para generar el versionamiento de los releases.

    Esto permite que en `main` se genere un release estable en donde el paquete se publica en GitHub Packages, el release queda visible, el tag y así mismo el el paquete

    Por otro lado, en las ramas `feature/tu-nombre` y `feature/tu-nombre-v2` se genera un release beta o alpha.

3. En el archivo anterior `.releaserc`, cambia los nombres de las ramas para que concidan con las creadas en los laboratorios previos.
4. Agregar los siguientes `scripts` en el `package.json`:

    ```json
    "scripts": {
      "test": "jest",
      "build": "rm -rf dist && mkdir dist && cp src/math.js dist/",
      "release": "semantic-release"
    },
    ```

5. En la carpeta `.github/workflows` crear un workflow `release.yml` y copiar el contenido de `release.example.yml`.
6. En `release.example.yml` crear comentarios (usando `#`) y explicar cada una de las siguientes secciones:

    - [ ] on
    - [ ] runs-on
    - [ ] permissions

    Los pasos (steps):

    - [ ] Checkout
    - [ ] Setup Node
    - [ ] Install dependencies
    - [ ] Build project
    - [ ] Create ZIP
    - [ ] Pack npm
    - [ ] Semantic Release

7. Eliminar el tag creado en el laboratorio anterior para evitar conflictos:

    Eliminar tag de la máquina local:

    ```bash
    git tag --delete v1.0.0-tu-nombre
    ```

    Eliminar tag del repositorio remoto:

    ```bash
    git push --delete origin v1.0.0-tu-nombre
    ```

8. Guardar cambios con el siguiente commit:

    ```bash
    git commit -m "feat: crear release.yml"
    ```

9. Subir cambios.
10. Esperar que ejecute el pipeline
11. Visualizar el release y paquete creado como beta.

## Laboratorio 10 - Automatización de un release estable

1. Hacer pull request desde tu rama `feature/tu-nombre` hacia `main`.
2. Observar el pipeline y sus logs.
3. En tu rama `feature/tu-nombre`, en `src\math.js` al final del archivo agregar una nueva función y actualizar el export:

    ```javascript
    function multiply(a, b) {
      return a * b;
    }

    module.exports = { sum, divide, multiply };
    ```

4. En `tests\math.test.js` al final del archivo agregar un nuevo test:

    ```javascript
    test('multiply works', () => {
      expect(multiply(4, 5)).toBe(20);
    });
    ```

5. Subir cambios con el commit:

    ```bash
    git commit -m "feat: add multiply function and corresponding tests"
    ```

6. Observar el nuevo release beta y su paquete correspondiente.
7. Desde tu rama `feature/tu-nombre` intentar hacer pull request hacia `main`.
8. Resolver conflictos, volver a subir cambios y hacer pull request.
9. Visualizar los releases y paquetes creados.
