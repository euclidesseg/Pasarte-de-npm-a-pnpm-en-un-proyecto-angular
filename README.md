# Cómo Migrar un Proyecto Angular +20 de npm a pnpm

Guía completa para:

* Instalar pnpm
* Usar pnpm por defecto
* Configurar Angular para pnpm
* Migrar proyectos Angular existentes
* Bloquear npm
* Mejorar seguridad con approve-builds
* Solucionar problemas comunes con `$PROFILE` en PowerShell

---

# Requisitos previos

## Verificar Node.js

```bash
node -v
npm -v
```

---

# Instalar pnpm

## Método recomendado (Corepack)

Habilitar Corepack:

```bash
corepack enable
```

Activar pnpm:

```bash
corepack prepare pnpm@latest --activate
```

Verificar:

```bash
pnpm -v
```

---

## Método alternativo usando npm

```bash
npm install -g pnpm
```

---

# Solución para Windows

Si aparece:

```text
ERR_PNPM_NO_GLOBAL_BIN_DIR
```

Ejecutar:

```bash
pnpm setup
```

Después:

1. Cerrar la terminal.
2. Abrir nuevamente la terminal.
3. Verificar:

```bash
pnpm -v
```

---

# Instalar Angular CLI

## Globalmente

```bash
pnpm add -g @angular/cli
```

Verificar:

```bash
ng version
```

---

# Configurar Angular para usar pnpm por defecto

Angular usa npm por defecto si no se configura manualmente.

Configurar:

```bash
ng config -g cli.packageManager pnpm
```

Verificar:

```bash
ng config -g cli.packageManager
```

Resultado esperado:

```text
pnpm
```

Ahora todos los nuevos proyectos:

```bash
ng new mi-app
```

usarán automáticamente:

* pnpm
* `pnpm-lock.yaml`
* NO crearán `package-lock.json`

---

# Crear un nuevo proyecto Angular con pnpm

```bash
ng new mi-app
```

Entrar al proyecto:

```bash
cd mi-app
```

---

# Migrar un proyecto Angular existente de npm a pnpm

Si el proyecto fue creado antes de configurar Angular para pnpm, debes migrarlo manualmente.

---

## 1. Cambiar packageManager en package.json

Buscar:

```json
"packageManager": "npm@10.x.x"
```

Cambiar por:

```json
"packageManager": "pnpm@11.3.0"
```

Usa tu versión real:

```bash
pnpm -v
```

---

## 2. Eliminar node_modules y package-lock.json

### PowerShell

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
```

### CMD

```cmd
rmdir /s /q node_modules
del package-lock.json
```

---

## 3. Configurar scripts seguros

```bash
pnpm config set ignore-scripts false --location=project
```

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

# approve-builds y seguridad

Después de instalar dependencias es normal ver:

```text
Ignored build scripts:
esbuild
@parcel/watcher
```

pnpm bloquea scripts sensibles automáticamente.

---

## Aprobar scripts necesarios

```bash
pnpm approve-builds
```

Normalmente Angular usa:

* esbuild
* @parcel/watcher
* lmdb
* msgpackr-extract

---

## Aplicar scripts aprobados

```bash
pnpm rebuild
```

---

# Configuración generada

pnpm puede generar:

```yaml
allowBuilds:
  '@parcel/watcher': true
  esbuild: true
  lmdb: true
  msgpackr-extract: true

ignoreScripts: false
```

---

# Explicación de ignoreScripts

## ignoreScripts: false

NO significa:

> ejecutar todo automáticamente

Significa:

> scripts permitidos, pero protegidos por approve-builds

---

## ignoreScripts: true

Bloquea:

* scripts de dependencias
* scripts lifecycle
* builds

Angular normalmente fallará.

---

# Ejecutar Angular

```bash
pnpm ng serve -o
```

o:

```bash
pnpm exec ng serve -o
```

---

# Configurar pnpm en proyectos Node.js

Agregar en package.json:

```json
"packageManager": "pnpm@11.3.0"
```

---

# Equivalencias npm → pnpm

| npm           | pnpm         |
| ------------- | ------------ |
| npm install   | pnpm install |
| npm run dev   | pnpm dev     |
| npm run build | pnpm build   |
| npm start     | pnpm start   |
| npx           | pnpm dlx     |

---

# Usar pnpm por defecto en PowerShell

## Verificar que estás usando PowerShell

Debes ver algo similar a:

```text
PS C:\Users\TuUsuario>
```

Si ves:

```text
C:\Users\TuUsuario>
```

estás usando CMD (Símbolo del sistema).

Los comandos `$PROFILE`, `Test-Path` y `. "$PROFILE"` solamente funcionan en PowerShell.

---

## Verificar la ruta del perfil

Mostrar la ubicación esperada:

```powershell
$PROFILE
```

Ejemplo:

```text
C:\Users\TuUsuario\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

---

## Verificar si el archivo existe

```powershell
Test-Path $PROFILE
```

Resultado:

```text
True
```

El archivo existe.

o

```text
False
```

El archivo aún no existe.

---

## Crear el archivo de perfil

Si `Test-Path $PROFILE` devuelve `False`:

```powershell
New-Item -ItemType File -Path $PROFILE -Force
```

Si la carpeta tampoco existe:

```powershell
New-Item -ItemType Directory -Path (Split-Path $PROFILE) -Force
New-Item -ItemType File -Path $PROFILE -Force
```

---

## Verificar que el archivo fue creado

```powershell
Get-Item $PROFILE
```

---

## Abrir el perfil

```powershell
notepad $PROFILE
```

Agregar:

```powershell
function npm {
    pnpm @args
}
```

Guardar y cerrar.

---

## Problemas con la política de ejecución

Si aparece un error similar a:

```text
running scripts is disabled on this system
```

Consultar:

```powershell
Get-ExecutionPolicy
```

Si devuelve:

```text
Restricted
```

Permitir scripts del usuario actual:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Confirmar con:

```text
Y
```

---

## Recargar el perfil

Usar:

```powershell
. "$PROFILE"
```

También puedes cerrar y abrir nuevamente PowerShell.

---

## Verificar que la función fue cargada

```powershell
Get-Command npm
```

Resultado esperado:

```text
CommandType Name
----------- ----
Function    npm
```

Ver implementación:

```powershell
Get-Command npm | Select-Object -ExpandProperty Definition
```

Resultado esperado:

```text
pnpm @args
```

Ahora:

```bash
npm install
```

ejecutará realmente:

```bash
pnpm install
```

---

# Bloquear completamente npm

Abrir nuevamente:

```powershell
notepad $PROFILE
```

Agregar:

```powershell
function npm {
    Write-Host "❌ npm está bloqueado. Usa pnpm." -ForegroundColor Red
}
```

Recargar:

```powershell
. "$PROFILE"
```

---

# Verificar PowerShell

Debe verse:

```text
PS C:\Users\TuUsuario>
```

Si ves:

```text
C:\Users\TuUsuario>
```

estás usando CMD.

---

# Seguridad adicional: permitir solo pnpm

En package.json:

```json
"scripts": {
  "preinstall": "npx only-allow pnpm"
}
```

Instalar:

```bash
pnpm add -D only-allow
```

Ahora:

```bash
npm install
```

fallará automáticamente.

---

# Qué subir a GitHub

## Sí subir

* pnpm-lock.yaml
* pnpm-workspace.yaml
* .npmrc

---

## NO subir

```text
node_modules
```

---

# Resultado final

Con esta configuración obtienes:

* Angular funcionando con pnpm.
* pnpm como gestor por defecto.
* npm bloqueado o redirigido.
* Instalaciones más rápidas.
* Menor uso de disco.
* Mayor seguridad.
* Control granular de scripts.
* Configuración moderna para equipos y CI/CD.
* Solución para equipos donde `$PROFILE` no existe inicialmente.
