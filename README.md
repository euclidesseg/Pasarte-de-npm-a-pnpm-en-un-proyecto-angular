# Cómo Migrar un proyecto Angular +20 de npm a pnpm

Guía completa para:

- instalar pnpm
- usar pnpm por defecto
- configurar Angular para pnpm
- migrar proyectos Angular existentes
- bloquear npm
- mejorar seguridad con approve-builds

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

# Método alternativo usando npm

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

1. Cerrar terminal
2. Abrir nuevamente
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

- pnpm
- `pnpm-lock.yaml`
- NO crearán `package-lock.json`

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

- esbuild
- @parcel/watcher
- lmdb
- msgpackr-extract

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

- scripts de dependencias
- scripts lifecycle
- builds

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

| npm | pnpm |
|---|---|
| npm install | pnpm install |
| npm run dev | pnpm dev |
| npm run build | pnpm build |
| npm start | pnpm start |
| npx | pnpm dlx |

---

# Usar pnpm por defecto en PowerShell

Abrir:

```powershell
notepad $PROFILE
```

Agregar:

```powershell
function npm {
    pnpm @args
}
```

Guardar.

Recargar perfil:

```powershell
. $PROFILE
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
. $PROFILE
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

- pnpm-lock.yaml
- pnpm-workspace.yaml
- .npmrc

---

## NO subir

```text
node_modules
```

---

# Resultado final

Con esta configuración obtienes:

- Angular funcionando con pnpm
- pnpm como gestor por defecto
- npm bloqueado o redirigido
- instalaciones más rápidas
- menor uso de disco
- mayor seguridad
- control granular de scripts
- configuración moderna para equipos y CI/CD
