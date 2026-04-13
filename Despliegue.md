# Proceso de Despliegue — Pokédex Angular en Azure Web Apps

## Información General
| Campo | Valor |
|---|---|
| Aplicación | Pokédex Angular |
| Repositorio fuente | github.com/rcuello/ac4dem1a |
| Ruta del proyecto | sistemas-distribuidos/poke-dex-lab/source/pokedex-angular |
| Plataforma | Microsoft Azure |
| Servicio | Azure Web Apps |
| Sistema operativo | Windows |
| Runtime | .NET 9 (STS) |
| Plan | Free F1 |
| Región | East US |

---

## Paso 1: Creación del recurso Web App en Azure

1. Ingresar al [Portal de Azure](https://portal.azure.com)
2. Buscar y seleccionar **"Web App"** en el Marketplace
3. Hacer clic en **"Crear"**
4. Completar la configuración:

```
Suscripción:        Azure for Students
Grupo de recursos:  rg-listrace-prod (nuevo)
Nombre:             app-listrace-web-prod-[INICIAL]
Publicar:           Código
Runtime stack:      .NET 9 (STS)
Sistema operativo:  Windows
Región:             East US
Plan de App:        Free F1 (Shared Infrastructure)
```

5. Hacer clic en **"Revisar y Crear"** → **"Crear"**
6. Esperar 2-3 minutos hasta ver: **"Your deployment is complete"**

---

## Paso 2: Preparación del código fuente

### 2.1 Descarga del repositorio
```bash
# Descargar ZIP desde GitHub
# URL: https://github.com/rcuello/ac4dem1a
# Ruta: sistemas-distribuidos/poke-dex-lab/source/pokedex-angular
```

### 2.2 Instalación de dependencias
```bash
cd pokedex-angular
npm install
```

### 2.3 Compilación para producción
```bash
npm run build -- --configuration production
```

**Salida esperada:**
```
✔ Browser application bundle generation complete.
✔ Copying assets complete.
✔ Index html generation complete.
```

> ⚠️ Los warnings de budget size no son errores, la compilación es exitosa.

### 2.4 Ubicación del build generado
```
dist/pokedex-angular/   ← estos archivos se suben a Azure
├── index.html
├── main.[hash].js
├── styles.[hash].css
├── assets/
│   ├── images/
│   └── fonts/
└── ...
```

---

## Paso 3: Despliegue con Kudu

### 3.1 Acceder a Kudu
1. Portal de Azure → Web App → **Development Tools** → **Advanced Tools**
2. Clic en **"Go →"**
3. Kudu → **Debug Console** → **CMD**

### 3.2 Navegar a wwwroot y subir archivos
```cmd
cd site\wwwroot
```
Arrastrar todo el contenido de `dist/pokedex-angular/` al explorador de Kudu.

### 3.3 Crear estructura para assets
```cmd
mkdir pokedex-angular
move assets pokedex-angular\assets
```

---

## Paso 4: Configuración de seguridad (web.config)

Crear `web.config` en `/site/wwwroot/`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <httpProtocol>
      <customHeaders>
        <add name="X-Frame-Options" value="SAMEORIGIN" />
        <add name="X-Content-Type-Options" value="nosniff" />
        <add name="X-XSS-Protection" value="1; mode=block" />
        <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
        <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
        <add name="Permissions-Policy" value="geolocation=(), microphone=(), camera=()" />
        <add name="Content-Security-Policy" value="default-src 'self';
          script-src 'self' 'unsafe-inline' 'unsafe-eval';
          style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
          style-src-elem 'self' 'unsafe-inline' https://fonts.googleapis.com;
          img-src 'self' data: https:;
          font-src 'self' data: https://fonts.gstatic.com;
          connect-src 'self' https://pokeapi.co https://beta.pokeapi.co;" />
      </customHeaders>
    </httpProtocol>
    <rewrite>
      <rules>
        <rule name="Angular Routes" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

**Resultado en securityheaders.com: Calificación A ✅**

---

## Errores encontrados y soluciones

### Error 1: Imágenes no cargan (404)
**Síntoma:**
```
GET /assets/images/red-walking-new.gif 404 (Not Found)
```
**Causa:** `environment.prod.ts` tenía `imagesPath: '/pokedex-angular/assets/images'`

**Solución:** Editar `src/environments/environment.prod.ts`:
```typescript
// Antes
imagesPath: '/pokedex-angular/assets/images',
// Después
imagesPath: '/assets/images',
```
Recompilar con `npm run build -- --configuration production`

---

### Error 2: Rutas relativas en imágenes del home
**Síntoma:** `<img src="./assets/images/red-walking-old.gif" />` falla con el router de Angular.

**Solución:** Cambiar en `home.component.html`:
```html
<!-- Antes -->
<img src="./assets/images/red-walking-old.gif" />
<!-- Después -->
<img src="/assets/images/red-walking-old.gif" />
```

---

### Error 3: Conflicto 409 al crear carpetas en Kudu
**Síntoma:**
```
Conflicto 409: El recurso representa un directorio que no puede actualizarse.
```
**Causa:** Kudu no permite crear carpetas vacías manualmente.

**Solución:** Arrastrar la carpeta con archivos directamente, o usar:
```cmd
mkdir pokedex-angular
xcopy site\wwwroot\assets site\wwwroot\pokedex-angular\assets /E /I /Y
```

---

### Error 4: CSP bloqueando Google Fonts
**Síntoma:**
```
Loading stylesheet 'https://fonts.googleapis.com' violates Content Security Policy
```
**Solución:** Agregar al CSP en `web.config`:
```
style-src ... https://fonts.googleapis.com
style-src-elem ... https://fonts.googleapis.com
font-src ... https://fonts.gstatic.com
```

---

## Validación final

| Verificación | Estado |
|---|---|
| App carga en Azure | ✅ |
| Lista de Pokémon visible | ✅ |
| Detalles de Pokémon funcionan | ✅ |
| Assets e imágenes cargan | ✅ |
| Sin errores en consola | ✅ |
| Calificación securityheaders.com | ✅ A |
| HTTPS activo | ✅ |
| Routing Angular funciona | ✅ |
