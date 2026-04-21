# Proceso de Despliegue — Pokédex Angular en Azure Static Web Apps

## Información General

| Campo | Valor |
|---|---|
| Aplicación | Pokédex Angular |
| Repositorio fuente | github.com/rcuello/ac4dem1a |
| Ruta del proyecto | sistemas-distribuidos/poke-dex-lab/source/pokedex-angular |
| Plataforma | Microsoft Azure |
| Servicio | Azure Static Web Apps |
| Plan | Free |
| Región | East US 2 |
| URL pública | https://zealous-pond-060bcfe10.7.azurestaticapps.net/ |

---

## Paso 1: Creación del recurso en Azure

1. Ingresar al [Portal de Azure](https://portal.azure.com)
2. Buscar y seleccionar **"Static Web Apps"** en el Marketplace
3. Hacer clic en **"Crear"**
4. Completar la configuración:

```
Suscripción:        Azure for Students
Grupo de recursos:  rg-pokedex-prod (nuevo)
Nombre:             pokedex-app
Plan:               Free
Región:             East US 2
Fuente:             GitHub
```

5. Conectar con GitHub y seleccionar el repositorio `pokedex`
6. Configurar el build:

```
Ubicación de la aplicación:  /
Ubicación de la API:         (vacío)
Ubicación de salida:         dist/pokedex-angular
```

7. Hacer clic en **"Revisar y Crear"** → **"Crear"**
8. Azure genera automáticamente un workflow en `.github/workflows/`

---

## Paso 2: Corrección del workflow de GitHub Actions

Al ejecutarse el workflow por primera vez, falló por un paso de **Codecov** que no era necesario para el despliegue. Se comentó el paso en el archivo `.github/workflows/azure-static-web-apps-*.yml`:

```yaml
# - name: Upload coverage to Codecov
#   run: ./node_modules/.bin/codecov
```

---

## Paso 3: Corrección de rutas de imágenes

**Síntoma:**
```
GET /pokedex-angular/assets/images/red-walking-new.gif 404 (Not Found)
```

**Causa:** El archivo `src/environments/environment.prod.ts` tenía una ruta incorrecta:

```typescript
// Antes (incorrecto)
imagesPath: '/pokedex-angular/assets/images',

// Después (correcto)
imagesPath: '/assets/images',
```

También se corrigieron rutas relativas en `home.component.html`:

```html
<!-- Antes -->
<img src="./assets/images/red-walking-old.gif" />

<!-- Después -->
<img src="/assets/images/red-walking-old.gif" />
```

---

## Paso 4: Configuración del enrutamiento Angular

Azure Static Web Apps no conoce las rutas de Angular por defecto y devuelve 404 al navegar directamente a una ruta. Se creó el archivo `staticwebapp.config.json` en la raíz del proyecto con la regla de fallback:

```json
{
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/images/.{png,jpg,gif,svg}", "/css/", "/js/*"]
  }
}
```

---

## Paso 5: Configuración de seguridad (Encabezados HTTP)

Se añadieron encabezados HTTP de seguridad al mismo archivo `staticwebapp.config.json` para obtener calificación **A+** en securityheaders.com:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; style-src-elem 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; img-src 'self' data: https://raw.githubusercontent.com https://pokeapi.co https://assets.pokemon.com https://beta.pokeapi.co; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co https://beta.pokeapi.co/graphql/v1beta; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer",
    "Permissions-Policy": "camera=(), microphone=(), geolocation=(), payment=()",
    "X-XSS-Protection": "1; mode=block"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/images/.{png,jpg,gif,svg}", "/css/", "/js/*"]
  }
}
```

---

## Errores encontrados y soluciones

### Error 1: Codecov falla en el workflow

**Síntoma:**
```
TypeError: result.split is not a function
Error: Process completed with exit code 1.
```

**Causa:** La versión de Codecov incluida en el proyecto es incompatible con Node.js 18+.

**Solución:** Comentar el paso de Codecov en el archivo `.yml` del workflow ya que no es necesario para el despliegue.

---

### Error 2: Carpeta de salida no encontrada

**Síntoma:**
```
The app build failed to produce artifact folder: 'dist/pokedex'.
```

**Causa:** El nombre del proyecto en `package.json` es `pokedex-angular`, no `pokedex`, por lo que Angular genera el build en `dist/pokedex-angular`.

**Solución:** Cambiar `output_location` en el workflow y en la configuración de Azure a `dist/pokedex-angular`.

---

### Error 3: Imágenes no cargan en producción (404)

**Síntoma:**
```
GET /pokedex-angular/assets/images/red-walking-new.gif 404 (Not Found)
```

**Causa:** `environment.prod.ts` tenía `imagesPath: '/pokedex-angular/assets/images'`.

**Solución:** Corregir la ruta a `/assets/images` y recompilar.

---

### Error 4: CSP bloquea PokeAPI y Google Fonts

**Síntoma:**
```
Connecting to 'https://beta.pokeapi.co/graphql/v1beta' violates Content Security Policy
Loading stylesheet 'https://fonts.googleapis.com' violates Content Security Policy
```

**Solución:** Ampliar el CSP para permitir explícitamente los dominios externos usados por la app:
- `connect-src` → `https://pokeapi.co https://beta.pokeapi.co`
- `style-src` y `style-src-elem` → `https://fonts.googleapis.com`
- `font-src` → `https://fonts.gstatic.com`

---

### Error 5: Rutas de Angular devuelven 404 en Azure

**Síntoma:**
```
GET https://zealous-pond-060bcfe10.7.azurestaticapps.net/error 404 (Not Found)
```

**Causa:** Azure no redirige las rutas desconocidas al `index.html` por defecto.

**Solución:** Agregar `navigationFallback` en `staticwebapp.config.json`.

---

## Validación Final

| Verificación | Estado |
|---|---|
| App carga en Azure | ✅ |
| Lista de Pokémon visible | ✅ |
| Detalles de Pokémon funcionan | ✅ |
| Assets e imágenes cargan | ✅ |
| Sin errores en consola | ✅ |
| Calificación securityheaders.com | ✅ A+ |
| HTTPS activo | ✅ |
| Routing Angular funciona | ✅ |
