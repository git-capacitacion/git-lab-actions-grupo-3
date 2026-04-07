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
    width:
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
