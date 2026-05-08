# Web Apps PaaS — Lab 09a

> Fase: F2 | Cert: AZ-104 | Lab: 09a | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 09a
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.21 USD/hora (App Service Plan Premium V3 P1V3)
**Región usada:** Central US (East US puede tener cuota agotada)
**Duración estimada:** 45 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Crear y configurar Azure Web App](#tarea-1)
- [Tarea 2 — Crear deployment slot de staging](#tarea-2)
- [Tarea 3 — Configurar despliegue continuo desde GitHub](#tarea-3)
- [Tarea 4 — Swap entre staging y producción](#tarea-4)
- [Tarea 5 — Configurar autoscaling](#tarea-5)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Crear una Azure Web App con App Service Plan Premium y configurar su runtime
- [ ] Crear deployment slots para pruebas previas a producción
- [ ] Configurar despliegue continuo desde un repositorio Git externo
- [ ] Realizar un swap entre staging y producción sin downtime
- [ ] Configurar autoscaling automático para responder a la demanda

**Habilidades del examen que cubre este lab:**
> **Deploy and manage Azure compute resources (20–25%)**
> — Configure Azure App Service
> — Configure deployment slots
> — Configure autoscaling

---

## ✅ Prerequisitos

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- No se necesitan labs anteriores para este lab

**Conocimientos previos:**
- PaaS vs IaaS: en PaaS no administras el SO ni el servidor web — Azure lo gestiona
- Deployment slots: entornos paralelos (staging, QA) que comparten el App Service Plan pero tienen URL y configuración independiente

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| App Service Plan Premium V3 P1V3 | Plataforma de cómputo para la Web App | ~$0.172/hora |
| Azure Web App | Aplicación PHP Hello World | Incluido en plan |
| Deployment Slot (staging) | Entorno de pruebas | Incluido en plan Premium |
| Azure Load Testing | Prueba de carga para validar autoscaling | ~$0.10/VUH |

---

## 🏗️ Arquitectura del lab

```
App Service Plan: Premium V3 P1V3 (Central US)
│
├── Web App: <nombre-unico> (producción)
│   ├── URL: https://<nombre>.azurewebsites.net
│   ├── Runtime: PHP 8.2 / Linux
│   └── Source: GitHub (php-docs-hello-world) ← después del swap
│
└── Deployment Slot: staging
    ├── URL: https://<nombre>-staging.azurewebsites.net
    └── Source: https://github.com/Azure-Samples/php-docs-hello-world
```

---

## Tarea 1 — Crear y configurar Azure Web App

**Objetivo de la tarea:** Crear la Web App con su App Service Plan. El plan determina los recursos de cómputo disponibles — el Premium V3 P1V3 es necesario para deployment slots y autoscaling.

### Método A — Portal web

1. Portal → **App Services** → **+ Crear** → **Web App**

   <img src="capturas/t1-paso-01-buscar-app-services.png" alt="Buscar App Services" width="50%">
   <img src="capturas/t1-paso-02-app-services-overview.png" alt="App Services overview" width="50%">
   <img src="capturas/t1-paso-03-crear-web-app.png" alt="Crear Web App" width="50%">

2. Configura **Basics**:
   - **RG:** `az104-rg9` | **Name:** nombre único global
   - **Publish:** Code | **Runtime stack:** PHP 8.2 | **OS:** Linux
   - **Region:** East US (si hay error de cuota, usa Central US)
   - **Pricing plan:** Premium V3 P1V3

   <img src="capturas/t1-paso-04-basics-webapp.png" alt="Basics WebApp" width="50%">

3. Si hay error de cuota en la región:

   <img src="capturas/t1-paso-05-error-cuota-region.png" alt="Error cuota región" width="50%">
   <img src="capturas/t1-paso-06-error-detalle.png" alt="Error detalle" width="50%">

4. Cambia a **Central US** y vuelve a intentar

   <img src="capturas/t1-paso-07-cambiar-region.png" alt="Cambiar región" width="50%">

5. **Review + create** → **Create** → **Go to resource**

   <img src="capturas/t1-paso-08-webapp-desplegada.png" alt="WebApp desplegada" width="50%">

### Método B — CLI

```powershell
az group create --name "az104-rg9" --location "centralus"

# App Service Plan Premium V3
az appservice plan create `
  --name "plan-lab09a" `
  --resource-group "az104-rg9" `
  --location "centralus" `
  --sku "P1V3" `
  --is-linux

# Web App PHP
$appName = "webapp-lab09a-$(Get-Random -Maximum 9999)"
az webapp create `
  --name $appName `
  --resource-group "az104-rg9" `
  --plan "plan-lab09a" `
  --runtime "PHP:8.2"
```

**Resultado esperado:**
> Web App creada con URL `https://<nombre>.azurewebsites.net` mostrando la página por defecto de Azure App Service.

---

## Tarea 2 — Crear deployment slot de staging

**Objetivo de la tarea:** Los deployment slots permiten probar cambios en un entorno idéntico al de producción antes de publicarlos. El swap es instantáneo y reversible.

### Método A — Portal web

1. Desde la Web App → **Default domain** → verifica la página por defecto

   <img src="capturas/t2-paso-01-default-domain.png" alt="Default domain" width="50%">
   <img src="capturas/t2-paso-02-pagina-default.png" alt="Página default" width="50%">

2. **Deployment → Deployment slots** → **+ Add slot**:
   - **Name:** `staging` | **Clone settings from:** Do not clone settings

   <img src="capturas/t2-paso-03-deployment-slots-menu.png" alt="Deployment slots menú" width="50%">
   <img src="capturas/t2-paso-04-add-slot-staging.png" alt="Add slot staging" width="50%">
   <img src="capturas/t2-paso-05-slot-configurado.png" alt="Slot configurado" width="50%">

3. Refresca → verifica que aparecen **Production** y **Staging**

   <img src="capturas/t2-paso-06-slots-lista.png" alt="Slots lista" width="50%">

4. Selecciona el slot **Staging** → verifica que tiene URL diferente a producción

   <img src="capturas/t2-paso-07-staging-slot-detalle.png" alt="Staging slot detalle" width="50%">

**Resultado esperado:**
> Slot `staging` creado con URL `https://<nombre>-staging.azurewebsites.net`.

---

## Tarea 3 — Configurar despliegue continuo desde GitHub

**Objetivo de la tarea:** Conectar el slot de staging a un repositorio GitHub público. Cada push al branch configurado despliega automáticamente en staging.

### Método A — Portal web

1. Desde el slot **Staging** → **Deployment Center** → **Settings**

   <img src="capturas/t3-paso-01-deployment-center.png" alt="Deployment Center" width="50%">

2. **Source:** External Git → configura:
   - **Repository:** `https://github.com/Azure-Samples/php-docs-hello-world`
   - **Branch:** `master`
   → **Save**

   <img src="capturas/t3-paso-02-external-git-config.png" alt="External Git config" width="50%">

3. Espera 1-2 minutos → navega al **Default domain** del slot staging → verifica **Hello World!**

   <img src="capturas/t3-paso-03-hello-world-staging.png" alt="Hello World staging" width="50%">

**Resultado esperado:**
> El slot de staging muestra la aplicación "Hello World" desplegada desde GitHub, mientras producción sigue mostrando la página por defecto.

---

## Tarea 4 — Swap entre staging y producción

**Objetivo de la tarea:** El swap intercambia los contenidos de staging y producción de forma instantánea. Si algo falla, puedes hacer swap de vuelta. Es la forma estándar de publicar cambios en producción con zero-downtime.

### Método A — Portal web

1. **Deployment slots** → **Swap**

   <img src="capturas/t4-paso-01-deployment-slots-swap.png" alt="Deployment slots swap" width="50%">

2. Revisa la configuración → **Start Swap**

   <img src="capturas/t4-paso-02-start-swap.png" alt="Start swap" width="50%">

3. Espera la notificación de swap completado

   <img src="capturas/t4-paso-03-swap-notificacion.png" alt="Swap notificación" width="50%">
   <img src="capturas/t4-paso-04-swap-completado.png" alt="Swap completado" width="50%">

4. Navega a la URL de **producción** → debe mostrar Hello World

   <img src="capturas/t4-paso-05-buscar-app-services.png" alt="Buscar App Services" width="50%">
   <img src="capturas/t4-paso-06-produccion-app.png" alt="Producción app" width="50%">
   <img src="capturas/t4-paso-07-hello-world-produccion.png" alt="Hello World producción" width="50%">

**Resultado esperado:**
> La URL de producción muestra ahora "Hello World!" — el código de staging está en producción.

---

## Tarea 5 — Configurar autoscaling

**Objetivo de la tarea:** Configurar el App Service Plan para que escale automáticamente cuando la demanda aumenta. Se valida con una prueba de carga.

### Método A — Portal web

1. **App Service plan → Scale out**

   <img src="capturas/t5-paso-01-scale-out-menu.png" alt="Scale out menú" width="50%">

2. **Scaling:** Automatic | **Maximum burst:** 2 → **Save**

   <img src="capturas/t5-paso-02-autoscale-automatic.png" alt="Autoscale automatic" width="50%">

3. **Diagnose and solve problems** → **Load Test your App** → **Create Load Test**

   <img src="capturas/t5-paso-03-diagnose-solve.png" alt="Diagnose solve" width="50%">
   <img src="capturas/t5-paso-04-create-load-test.png" alt="Create load test" width="50%">

4. Nombre único → **Review + create** → **Create** → **Go to resource**

   <img src="capturas/t5-paso-05-nuevo-load-test.png" alt="Nuevo load test" width="50%">
   <img src="capturas/t5-paso-06-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t5-paso-07-load-test-creado.png" alt="Load test creado" width="50%">

5. **Create** → **Add request** → introduce la URL de producción

   <img src="capturas/t5-paso-08-crear-request.png" alt="Crear request" width="50%">
   <img src="capturas/t5-paso-09-agregar-url.png" alt="Agregar URL" width="50%">
   <img src="capturas/t5-paso-10-request-agregado.png" alt="Request agregado" width="50%">

6. **Review + create** → **Create** → monitorea las métricas en tiempo real

   <img src="capturas/t5-paso-11-iniciar-test.png" alt="Iniciar test" width="50%">
   <img src="capturas/t5-paso-12-metricas-tiempo-real.png" alt="Métricas tiempo real" width="50%">
   <img src="capturas/t5-paso-13-resultado-test.png" alt="Resultado test" width="50%">

**Resultado esperado:**
> La prueba de carga genera tráfico y las métricas muestran usuarios virtuales, tiempo de respuesta y requests por segundo.

---

## ⚠️ Errores comunes

### Error 1 — `QuotaExceeded` al crear el App Service Plan

**Síntoma:** Error de cuota al intentar crear el plan Premium en East US.

**Causa:** La suscripción tiene límite de cores Premium por región.

**Solución:** Cambia a Central US, West US 2 o West Europe.

---

### Error 2 — El swap no refleja cambios

**Síntoma:** Después del swap, producción sigue mostrando la página por defecto.

**Causa:** El browser cachea la respuesta. Los CDN y proxies también pueden cachear.

**Solución:** Abre en ventana de incógnito o fuerza recarga con `Ctrl+Shift+R`.

---

## 🧹 Limpieza de recursos

> ⚠️ El App Service Plan Premium V3 cobra ~$0.172/hora incluso sin tráfico.

### Método A — Portal

`az104-rg9` → **Eliminar grupo de recursos**

<img src="capturas/limpieza-eliminar-rg.png" alt="Limpieza" width="50%">

### Método B — CLI

```powershell
az group delete --name "az104-rg9" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| App Service Plan Premium V3 P1V3 | ~1 hora | ~$0.172 |
| Azure Load Testing (50 VU × 10 min) | ~0.08 VUH | ~$0.008 |
| **Total lab** | | **~$0.18** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** Central US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| PaaS Web Apps | App Service | Elastic Beanstalk / Amplify | App Engine | OCI DevOps |
| Deployment slots | Sí (Premium+) | Sí (Beanstalk envs) | Sí (traffic splitting) | No nativo |
| Autoscaling | Sí (HTTP-based) | Sí | Sí | Sí |
| Costo plan equiv./hora | ~$0.172 | ~$0.16 (t3.medium) | ~$0.15 | ~$0.10 |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- App Service elimina la gestión de SO y servidor web — solo te ocupas del código
- Los deployment slots son la alternativa PaaS al Blue/Green deployment en IaaS
- El swap es instantáneo porque Azure solo cambia los punteros DNS internos
- El autoscaling de App Service responde a HTTP requests, no a CPU — escala más rápido que VMSS para cargas web

**Cómo se usa en labs futuros:**
> El patrón de despliegue con slots es equivalente al que se usa en entornos AKS con blue/green deployments — el **Lab-09c** muestra la versión containerizada.

**Siguiente lab recomendado:**
👉 [Lab 09b — Azure Container Instances](../../Lab-09b-ACI/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
