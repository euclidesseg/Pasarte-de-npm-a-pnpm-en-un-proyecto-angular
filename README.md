---

# Instalar pnpm y usarlo por defecto en todo el sistema

## Verificar Node.js

Antes de instalar pnpm, verifica que Node.js esté instalado correctamente:

```bash
node -v
npm -v
```

---

# Instalar pnpm globalmente

## Método recomendado (Corepack)

Node.js moderno incluye Corepack, que permite administrar gestores de paquetes como pnpm de forma oficial.

Habilitar Corepack:

```bash
corepack enable
```

Activar pnpm:

```bash
corepack prepare pnpm@latest --activate
```

Verificar instalación:

```bash
pnpm -v
```

---

# Método alternativo usando npm

También puedes instalar pnpm globalmente usando npm:

```bash
npm install -g pnpm
```

Verificar:

```bash
pnpm -v
```

---

# Solución para Windows: ERR_PNPM_NO_GLOBAL_BIN_DIR

Si aparece un error similar a:

```text
ERR_PNPM_NO_GLOBAL_BIN_DIR
```

Ejecuta:

```bash
pnpm setup
```

Después:

1. Cierra completamente la terminal.
2. Abre nuevamente CMD, PowerShell o VSCode.
3. Verifica:

```bash
pnpm -v
```

---

# Configurar pnpm como gestor por defecto en Angular

Angular usa npm por defecto si no se configura manualmente.

Configurar Angular CLI globalmente:

```bash
ng config -g cli.packageManager pnpm
```

Verificar configuración:

```bash
ng config -g cli.packageManager
```

Resultado esperado:

```text
pnpm
```

Con esto:

```bash
ng new mi-app
```

creará automáticamente proyectos usando:

- pnpm
- `pnpm-lock.yaml`
- SIN `package-lock.json`

---

# Configurar proyectos Node.js para usar pnpm

En cada proyecto se recomienda definir:

```json
"packageManager": "pnpm@10.15.1"
```

Puedes usar tu versión real:

```bash
pnpm -v
```

Ejemplo:

```json
"packageManager": "pnpm@11.3.0"
```

Esto permite que Corepack y otras herramientas detecten automáticamente que el proyecto usa pnpm.

---

# Comandos equivalentes npm → pnpm

| npm | pnpm |
|---|---|
| npm install | pnpm install |
| npm run dev | pnpm dev |
| npm run build | pnpm build |
| npm start | pnpm start |
| npm create vite | pnpm create vite |
| npx | pnpm dlx |

---

# Usar pnpm por defecto en PowerShell

## Redirigir npm automáticamente hacia pnpm

PowerShell permite sobrescribir el comando `npm` para usar pnpm automáticamente.

Abrir PowerShell y ejecutar:

```powershell
notepad $PROFILE
```

Si el archivo no existe, PowerShell preguntará si deseas crearlo.

Agregar al archivo:

```powershell
function npm {
    pnpm @args
}
```

Guardar el archivo.

Luego recargar el perfil:

```powershell
. $PROFILE
```

Ahora, cuando ejecutes:

```bash
npm install
```

PowerShell realmente ejecutará:

```bash
pnpm install
```

---

# Bloquear completamente npm en PowerShell

Si prefieres impedir el uso de npm:

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

Guardar y recargar:

```powershell
. $PROFILE
```

Ahora cualquier intento de usar npm mostrará:

```text
❌ npm está bloqueado. Usa pnpm.
```

---

# Verificar si PowerShell está activo

El comando:

```powershell
. $PROFILE
```

SOLO funciona en PowerShell.

Debes ver algo similar a:

```text
PS C:\Users\TuUsuario>
```

Si ves:

```text
C:\Users\TuUsuario>
```

entonces estás usando CMD y no PowerShell.

---

# Configuración recomendada moderna

La configuración recomendada actualmente para proyectos modernos es:

- Node.js
- Corepack
- pnpm
- `packageManager`
- `pnpm-lock.yaml`
- `only-allow pnpm` (opcional)
- bloqueo o redirección de npm en PowerShell

---

# Seguridad adicional: permitir solo pnpm en proyectos

Puedes impedir que un proyecto use npm agregando:

```json
"scripts": {
  "preinstall": "npx only-allow pnpm"
}
```

Instalar dependencia:

```bash
pnpm add -D only-allow
```

Ahora, si alguien ejecuta:

```bash
npm install
```

el proyecto fallará y exigirá pnpm.

---

# Resultado final

Con esta configuración obtienes:

- pnpm configurado globalmente.
- Angular usando pnpm por defecto.
- Node.js compatible con pnpm.
- npm redirigido o bloqueado.
- Instalaciones más rápidas.
- Menor consumo de disco.
- Mayor seguridad frente a scripts maliciosos.
- Configuración consistente para equipos y CI/CD.
