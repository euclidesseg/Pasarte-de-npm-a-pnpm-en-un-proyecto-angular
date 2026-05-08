# Cómo Migrar un proyecto Angular +20 de npm a pnpm

Guía completa para migrar un proyecto de Angular desde npm hacia pnpm de forma segura y moderna.

---

# Requisitos previos

## 1. Tener Node.js instalado

Verifica la instalación:

```bash
node -v
npm -v
```

---

## 2. Instalar pnpm globalmente

```bash
npm install -g pnpm
```

Verifica:

```bash
pnpm -v
```

---

# Configurar pnpm correctamente en Windows

Si aparece un error similar a:

```text
ERR_PNPM_NO_GLOBAL_BIN_DIR
```

Ejecuta:

```bash
pnpm setup
```

Luego:

1. Cierra completamente la terminal.
2. Abre nuevamente CMD, PowerShell o VSCode.
3. Verifica:

```bash
pnpm -v
```

---

# Instalar Angular CLI globalmente

Instalar Angular CLI:

```bash
pnpm add -g @angular/cli
```

Verifica:

```bash
ng version
```

---

# Configurar Angular para usar pnpm por defecto

IMPORTANTE:

Angular usa npm por defecto si no se configura explícitamente.

Por eso debes ejecutar:

```bash
ng config -g cli.packageManager pnpm
```

Verifica:

```bash
ng config -g cli.packageManager
```

Debe mostrar:

```text
pnpm
```

Con esto, todos los nuevos proyectos Angular creados con:

```bash
ng new mi-app
```

usarán automáticamente:

- pnpm
- `pnpm-lock.yaml`
- NO crearán `package-lock.json`

---

# Crear un nuevo proyecto Angular con pnpm

```bash
ng new mi-app
```

Luego:

```bash
cd mi-app
```

---

# Migrar un proyecto Angular existente de npm a pnpm

Si creaste el proyecto ANTES de configurar:

```bash
ng config -g cli.packageManager pnpm
```

entonces tu proyecto seguirá configurado para npm.

En ese caso debes migrarlo manualmente.

---

## 1. Cambiar `packageManager` en `package.json`

Busca algo como:

```json
"packageManager": "npm@10.x.x"
```

y cámbialo por:

```json
"packageManager": "pnpm@10.0.0"
```

O mejor aún, usa tu versión real de pnpm:

```bash
pnpm -v
```

Ejemplo:

```json
"packageManager": "pnpm@10.15.1"
```

---

## 2. Eliminar dependencias y lock file de npm

Eliminar:

- `node_modules`
- `package-lock.json`

### Linux / Mac

```bash
rm -rf node_modules
rm package-lock.json
```

### Windows PowerShell

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
```

---

## 3. Configurar manejo seguro de scripts

Configurar pnpm para permitir scripts, pero usando control granular con `approve-builds`.

```bash
pnpm config set ignore-scripts false --location=project
```

Esto crea configuración local para el proyecto.

---

## 4. Instalar dependencias con pnpm

```bash
pnpm install
```

Esto creará:

```text
pnpm-lock.yaml
```

---

# Scripts de construcción y seguridad

Después de `pnpm install`, es normal ver algo como:

```text
Ignored build scripts:
esbuild
@parcel/watcher
...
```

Esto significa que pnpm detectó scripts de construcción potencialmente sensibles y decidió NO ejecutarlos automáticamente.

---

## 5. Aprobar scripts necesarios

Ejecutar:

```bash
pnpm approve-builds
```

Aprobar los paquetes necesarios.

En Angular 21 normalmente son:

- `esbuild`
- `@parcel/watcher`
- `lmdb`
- `msgpackr-extract`

---

## 6. Archivo generado por pnpm

pnpm generará algo similar a:

```yaml
allowBuilds:
  '@parcel/watcher': true
  esbuild: true
  lmdb: true
  msgpackr-extract: true
ignoreScripts: false
```

---

# Explicación de la configuración

## `ignoreScripts: false`

NO significa:

> Ejecutar todos los scripts automáticamente.

Significa:

> Los scripts están permitidos.

Sin embargo:

- pnpm sigue protegiendo scripts sensibles.
- Los build scripts requieren aprobación mediante `approve-builds`.

---

## `allowBuilds`

Es una lista blanca de paquetes autorizados para ejecutar scripts de construcción.

Ejemplo:

```yaml
allowBuilds:
  esbuild: true
```

Esto permite que `esbuild` ejecute sus scripts.

---

## ¿Qué pasa con `ignoreScripts: true`?

Si configuras:

```yaml
ignoreScripts: true
```

Entonces:

- NO se ejecuta ningún script.
- Se bloquean scripts de dependencias.
- Se bloquean scripts lifecycle del proyecto.
- `allowBuilds` deja de tener efecto.

Esto suele romper Angular y otras herramientas modernas.

---

# Aplicar scripts aprobados

Después de aprobar scripts, ejecutar:

```bash
pnpm rebuild
```

Esto ejecuta los scripts previamente aprobados.

---

# Ejecutar Angular

Puedes iniciar Angular con:

```bash
pnpm ng serve -o
```

o:

```bash
pnpm exec ng serve -o
```

---

# Angular CLI global

NO es obligatorio instalar Angular CLI globalmente.

No necesitas:

```bash
npm install -g @angular/cli
```

Porque:

- Angular CLI ya existe dentro del proyecto.
- pnpm puede ejecutar el CLI local.

---

# Verificar configuración de scripts

Ver estado actual:

```bash
pnpm config get ignore-scripts
```

Resultados posibles:

| Resultado | Significado |
|---|---|
| `true` | Bloquea todos los scripts |
| `false` | Scripts permitidos |
| `undefined` | Usa valor por defecto (`false`) |

---

# Seguridad obtenida

Con esta configuración:

```yaml
allowBuilds:
  esbuild: true
ignoreScripts: false
```

Sucede lo siguiente:

- Los scripts aprobados sí se ejecutan.
- Los scripts de paquetes desconocidos NO se ejecutan automáticamente.
- pnpm pedirá aprobación antes de ejecutarlos.

---

# Qué scripts se ejecutan en `pnpm install`

## Dependencias aprobadas

Sí se ejecutan.

Ejemplo:

- `esbuild`
- `@parcel/watcher`

---

## Dependencias NO aprobadas

NO se ejecutan.

pnpm mostrará advertencias y pedirá aprobación.

---

## Scripts normales del proyecto

Ejemplo:

```json
"scripts": {
  "start": "ng serve"
}
```

NO se ejecutan automáticamente durante `pnpm install`.

Solo se ejecutan cuando el usuario los llama manualmente:

```bash
pnpm start
```

---

## Lifecycle scripts del proyecto

Ejemplo:

```json
"scripts": {
  "postinstall": "algo"
}
```

Sí pueden ejecutarse automáticamente durante `pnpm install`, siempre que:

```yaml
ignoreScripts: false
```

---

# Qué subir a GitHub

## Sí subir

- `pnpm-lock.yaml`
- `pnpm-workspace.yaml`
- `.npmrc` (si existe)

---

## NO subir

```text
node_modules
```

---

# Flujo recomendado a futuro

Cuando agregues nuevas dependencias:

```bash
pnpm install
```

Si pnpm detecta nuevos build scripts:

```bash
pnpm approve-builds
pnpm rebuild
```

---

# Configuración final recomendada

```yaml
allowBuilds:
  '@parcel/watcher': true
  esbuild: true
  lmdb: true
  msgpackr-extract: true
ignoreScripts: false
```

---

# Resultado final

Con esta configuración obtienes:

- Migración correcta de npm a pnpm.
- Angular 21 funcionando correctamente.
- Mayor seguridad frente a scripts maliciosos.
- Control granular sobre scripts de construcción.
- Configuración reproducible para equipos y GitHub.
- Instalaciones más rápidas y eficientes.# Migrar un proyecto Angular 21 de npm a pnpm

Guía completa para migrar un proyecto de Angular desde npm hacia pnpm de forma segura y moderna.

---

# Requisitos previos

## 1. Tener Node.js instalado

Verifica la instalación:

```bash
node -v
npm -v
```

---

## 2. Instalar pnpm globalmente

```bash
npm install -g pnpm
```

Verifica:

```bash
pnpm -v
```

---

# Configurar pnpm correctamente en Windows

Si aparece un error similar a:

```text
ERR_PNPM_NO_GLOBAL_BIN_DIR
```

Ejecuta:

```bash
pnpm setup
```

Luego:

1. Cierra completamente la terminal.
2. Abre nuevamente CMD, PowerShell o VSCode.
3. Verifica:

```bash
pnpm -v
```

---

# Instalar Angular CLI globalmente

Instalar Angular CLI:

```bash
pnpm add -g @angular/cli
```

Verifica:

```bash
ng version
```

---

# Configurar Angular para usar pnpm por defecto

IMPORTANTE:

Angular usa npm por defecto si no se configura explícitamente.

Por eso debes ejecutar:

```bash
ng config -g cli.packageManager pnpm
```

Verifica:

```bash
ng config -g cli.packageManager
```

Debe mostrar:

```text
pnpm
```

Con esto, todos los nuevos proyectos Angular creados con:

```bash
ng new mi-app
```

usarán automáticamente:

- pnpm
- `pnpm-lock.yaml`
- NO crearán `package-lock.json`

---

# Crear un nuevo proyecto Angular con pnpm

```bash
ng new mi-app
```

Luego:

```bash
cd mi-app
```

---

# Migrar un proyecto Angular existente de npm a pnpm

Si creaste el proyecto ANTES de configurar:

```bash
ng config -g cli.packageManager pnpm
```

entonces tu proyecto seguirá configurado para npm.

En ese caso debes migrarlo manualmente.

---

## 1. Cambiar `packageManager` en `package.json`

Busca algo como:

```json
"packageManager": "npm@10.x.x"
```

y cámbialo por:

```json
"packageManager": "pnpm@10.0.0"
```

O mejor aún, usa tu versión real de pnpm:

```bash
pnpm -v
```

Ejemplo:

```json
"packageManager": "pnpm@10.15.1"
```

---

## 2. Eliminar dependencias y lock file de npm

Eliminar:

- `node_modules`
- `package-lock.json`

### Linux / Mac

```bash
rm -rf node_modules
rm package-lock.json
```

### Windows PowerShell

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
```

---

## 3. Configurar manejo seguro de scripts

Configurar pnpm para permitir scripts, pero usando control granular con `approve-builds`.

```bash
pnpm config set ignore-scripts false --location=project
```

Esto crea configuración local para el proyecto.

---

## 4. Instalar dependencias con pnpm

```bash
pnpm install
```

Esto creará:

```text
pnpm-lock.yaml
```

---

# Scripts de construcción y seguridad

Después de `pnpm install`, es normal ver algo como:

```text
Ignored build scripts:
esbuild
@parcel/watcher
...
```

Esto significa que pnpm detectó scripts de construcción potencialmente sensibles y decidió NO ejecutarlos automáticamente.

---

## 5. Aprobar scripts necesarios

Ejecutar:

```bash
pnpm approve-builds
```

Aprobar los paquetes necesarios.

En Angular 21 normalmente son:

- `esbuild`
- `@parcel/watcher`
- `lmdb`
- `msgpackr-extract`

---

## 6. Archivo generado por pnpm

pnpm generará algo similar a:

```yaml
allowBuilds:
  '@parcel/watcher': true
  esbuild: true
  lmdb: true
  msgpackr-extract: true
ignoreScripts: false
```

---

# Explicación de la configuración

## `ignoreScripts: false`

NO significa:

> Ejecutar todos los scripts automáticamente.

Significa:

> Los scripts están permitidos.

Sin embargo:

- pnpm sigue protegiendo scripts sensibles.
- Los build scripts requieren aprobación mediante `approve-builds`.

---

## `allowBuilds`

Es una lista blanca de paquetes autorizados para ejecutar scripts de construcción.

Ejemplo:

```yaml
allowBuilds:
  esbuild: true
```

Esto permite que `esbuild` ejecute sus scripts.

---

## ¿Qué pasa con `ignoreScripts: true`?

Si configuras:

```yaml
ignoreScripts: true
```

Entonces:

- NO se ejecuta ningún script.
- Se bloquean scripts de dependencias.
- Se bloquean scripts lifecycle del proyecto.
- `allowBuilds` deja de tener efecto.

Esto suele romper Angular y otras herramientas modernas.

---

# Aplicar scripts aprobados

Después de aprobar scripts, ejecutar:

```bash
pnpm rebuild
```

Esto ejecuta los scripts previamente aprobados.

---

# Ejecutar Angular

Puedes iniciar Angular con:

```bash
pnpm ng serve -o
```

o:

```bash
pnpm exec ng serve -o
```

---

# Angular CLI global

NO es obligatorio instalar Angular CLI globalmente.

No necesitas:

```bash
npm install -g @angular/cli
```

Porque:

- Angular CLI ya existe dentro del proyecto.
- pnpm puede ejecutar el CLI local.

---

# Verificar configuración de scripts

Ver estado actual:

```bash
pnpm config get ignore-scripts
```

Resultados posibles:

| Resultado | Significado |
|---|---|
| `true` | Bloquea todos los scripts |
| `false` | Scripts permitidos |
| `undefined` | Usa valor por defecto (`false`) |

---

# Seguridad obtenida

Con esta configuración:

```yaml
allowBuilds:
  esbuild: true
ignoreScripts: false
```

Sucede lo siguiente:

- Los scripts aprobados sí se ejecutan.
- Los scripts de paquetes desconocidos NO se ejecutan automáticamente.
- pnpm pedirá aprobación antes de ejecutarlos.

---

# Qué scripts se ejecutan en `pnpm install`

## Dependencias aprobadas

Sí se ejecutan.

Ejemplo:

- `esbuild`
- `@parcel/watcher`

---

## Dependencias NO aprobadas

NO se ejecutan.

pnpm mostrará advertencias y pedirá aprobación.

---

## Scripts normales del proyecto

Ejemplo:

```json
"scripts": {
  "start": "ng serve"
}
```

NO se ejecutan automáticamente durante `pnpm install`.

Solo se ejecutan cuando el usuario los llama manualmente:

```bash
pnpm start
```

---

## Lifecycle scripts del proyecto

Ejemplo:

```json
"scripts": {
  "postinstall": "algo"
}
```

Sí pueden ejecutarse automáticamente durante `pnpm install`, siempre que:

```yaml
ignoreScripts: false
```

---

# Qué subir a GitHub

## Sí subir

- `pnpm-lock.yaml`
- `pnpm-workspace.yaml`
- `.npmrc` (si existe)

---

## NO subir

```text
node_modules
```

---

# Flujo recomendado a futuro

Cuando agregues nuevas dependencias:

```bash
pnpm install
```

Si pnpm detecta nuevos build scripts:

```bash
pnpm approve-builds
pnpm rebuild
```

---

# Configuración final recomendada

```yaml
allowBuilds:
  '@parcel/watcher': true
  esbuild: true
  lmdb: true
  msgpackr-extract: true
ignoreScripts: false
```

---

# Resultado final

Con esta configuración obtienes:

- Migración correcta de npm a pnpm.
- Angular 21 funcionando correctamente.
- Mayor seguridad frente a scripts maliciosos.
- Control granular sobre scripts de construcción.
- Configuración reproducible para equipos y GitHub.
- Instalaciones más rápidas y eficientes.
