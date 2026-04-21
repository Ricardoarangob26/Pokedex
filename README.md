# Pokédex Angular — Despliegue en Microsoft Azure

## Descripción

Aplicación web Pokédex desarrollada en Angular, desplegada en la nube pública de Microsoft Azure utilizando **Azure Static Web Apps** como servicio de hosting.

---

## ☁️ Creación de Cuenta en Azure for Students

### Requisitos previos

- Correo institucional universitario activo
- Acceso a internet

### Paso 1: Registro en Azure for Students

1. Ingresa a [Azure for Students](https://azure.microsoft.com/es-es/free/students/)
2. Haz clic en **"Empezar gratis"**
3. Inicia sesión con tu **correo institucional** (ej: usuario@universidad.edu.co)
4. Acepta los términos y condiciones de Microsoft
5. Completa la verificación de estudiante

### Paso 2: Activación de la suscripción

1. Una vez verificado, Azure asigna automáticamente:
   - **$100 USD** en créditos gratuitos
   - Acceso a servicios gratuitos por 12 meses
   - Servicios siempre gratuitos incluidos
2. Recibirás un correo de confirmación de Microsoft

### Paso 3: Acceso al Portal

1. Ve a [https://portal.azure.com](https://portal.azure.com)
2. Inicia sesión con tu cuenta institucional
3. Verifica que la suscripción **"Azure for Students"** aparezca activa en el panel principal

### Paso 4: Renovación de créditos (si es necesario)

1. Ingresa a [Azure for Students](https://azure.microsoft.com/es-es/free/students/)
2. Haz clic en **"Activar"**
3. Inicia sesión nuevamente con tu correo institucional
4. Sigue el proceso de renovación

---

## 🏗️ Arquitectura Utilizada

```
Internet → HTTPS → Azure Static Web Apps
                        ↓
              GitHub Actions (CI/CD automático)
                        ↓
              Resource Group (rg-pokedex-prod)
                        ↓
              Azure Subscription for Students
```

### Servicios de Azure usados

| Servicio | Nombre | Plan |
|---|---|---|
| Resource Group | rg-pokedex-prod | - |
| Static Web App | zealous-pond-060bcfe10 | Free |

---

## 🔗 URL de la aplicación

```
https://zealous-pond-060bcfe10.7.azurestaticapps.net/
```

---

## 🔒 Seguridad

La aplicación obtuvo calificación **A+** en [securityheaders.com](https://securityheaders.com), implementando los siguientes encabezados HTTP:

| Encabezado | Propósito |
|---|---|
| Content-Security-Policy | Previene XSS controlando recursos permitidos |
| Strict-Transport-Security | Fuerza el uso de HTTPS |
| X-Content-Type-Options | Evita detección automática de tipos de archivo |
| X-Frame-Options | Impide que la app se muestre en iframes externos |
| Referrer-Policy | Minimiza fuga de información en cabeceras HTTP |
| Permissions-Policy | Restringe acceso a APIs del navegador |
| X-XSS-Protection | Capa adicional contra ataques XSS |

---

## 💬 Reflexión Técnica

### ¿Qué vulnerabilidades previenen los encabezados implementados?

- **Content-Security-Policy** previene ataques XSS (Cross-Site Scripting) al controlar qué scripts, estilos y recursos externos puede cargar la aplicación.
- **Strict-Transport-Security** elimina ataques de downgrade de HTTPS a HTTP y ataques man-in-the-middle.
- **X-Frame-Options** previene ataques de clickjacking al bloquear que la app sea embebida en iframes de otros sitios.
- **X-Content-Type-Options** evita que el navegador adivine el tipo MIME de archivos, previniendo ejecución de código malicioso disfrazado.

### ¿Qué aprendiste sobre la relación entre despliegue y seguridad web?

El despliegue no termina cuando la aplicación es accesible públicamente. Configurar correctamente los encabezados HTTP es una capa fundamental de defensa que protege a los usuarios sin modificar el código fuente de la aplicación. Azure Static Web Apps facilita esto centralizando la configuración en un solo archivo `staticwebapp.config.json`.

### ¿Qué desafíos encontraste en el proceso?

- La variable `imagesPath` en `environment.prod.ts` apuntaba a una ruta incorrecta, lo que causaba que las imágenes no cargaran en producción.
- El Content-Security-Policy inicial era demasiado restrictivo y bloqueaba la PokeAPI, Google Fonts y scripts inline de Angular, requiriendo ajustes iterativos.
- El workflow de GitHub Actions incluía un paso de Codecov que fallaba y fue necesario desactivarlo para que el despliegue continuara.
