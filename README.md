# Pokédex Angular — Despliegue en Microsoft Azure

## Descripción
Aplicación web Pokédex desarrollada en Angular, desplegada en la nube pública de Microsoft Azure utilizando Azure Web Apps como servicio de hosting.

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
1. Ingresa a [Azure for Students Starter](https://azure.microsoft.com/es-es/free/students/)
2. Haz clic en **"Activar"**
3. Inicia sesión nuevamente con tu correo institucional
4. Sigue el proceso de renovación

---

## 🏗️ Arquitectura Utilizada

```
Internet → HTTPS → Azure Web App (app-listrace-web-prod)
                        ↓
              App Service Plan (Free F1)
                        ↓
              Resource Group (rg-listrace-prod)
                        ↓
              Azure Subscription for Students
```

### Servicios de Azure usados
| Servicio | Nombre | Plan |
|---|---|---|
| Resource Group | rg-listrace-prod | - |
| App Service Plan | ASP-rglistraceprod | Free F1 |
| Web App | app-listrace-web-prod | Free |

---

## 🔗 URL de la aplicación
```
https://app-listrace-web-prod-[raab-g8cmbmdeg0b7a5aw.mexicocentral-01].azurewebsites.net
```
