# Network Traffic Management — Lab 06

> Fase: F2 | Cert: AZ-104 | Lab: 06 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 06
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.85 USD/hora (3 VMs + LB + App Gateway)
**Región usada:** East US
**Duración estimada:** 50 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Provisionar infraestructura con plantilla ARM](#tarea-1)
- [Tarea 2 — Configurar Azure Load Balancer](#tarea-2)
- [Tarea 3 — Configurar Azure Application Gateway](#tarea-3)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Desplegar infraestructura base con plantillas ARM en segundos
- [ ] Configurar un Azure Load Balancer (capa 4) con backend pool, health probe y reglas de balanceo
- [ ] Configurar un Azure Application Gateway (capa 7) con path-based routing para imágenes y videos
- [ ] Validar que el tráfico se distribuye correctamente entre los backends

**Habilidades del examen que cubre este lab:**
> **Configure and manage virtual networking (25–30%)**
> — Configure load balancing
> — Configure Azure Application Gateway

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 04 — Virtual Networking *(VNets y subredes)*
- [ ] Lab 05 — Intersite Connectivity *(conceptos de tráfico entre redes)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- Archivos del lab descargados desde [Microsoft Learn AZ-104](https://microsoftlearning.github.io/AZ-104-MicrosoftAzureAdministrator/)

**Conocimientos previos:**
- Diferencia entre balanceo capa 4 (TCP/UDP) y capa 7 (HTTP/HTTPS con reglas de contenido)
- Health probes: el LB/AppGW solo envía tráfico a backends que responden al probe

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| 3× VM Standard_D2s_v3 | Backends para LB y App Gateway | ~$0.56/hora |
| Azure Load Balancer Standard | Balanceo capa 4 | ~$0.025/hora |
| Azure Application Gateway V2 | Balanceo capa 7 con path routing | ~$0.22/hora |
| VNet + subredes | Red del lab | $0.00 |

---

## 🏗️ Arquitectura del lab

```
Internet
│
├── Load Balancer (IP pública: az104-lbpip)     ← Tarea 2
│   └── Backend Pool: az104-be
│       ├── az104-06-vm0 (responde: Hello World from vm0)
│       └── az104-06-vm1 (responde: Hello World from vm1)
│
└── Application Gateway (IP pública: az104-gwpip)  ← Tarea 3
    └── Listener: HTTP:80
        ├── /image/* → az104-imagebe → az104-06-nic1
        ├── /video/* → az104-videobe → az104-06-nic2
        └── /* (default) → az104-appgwbe → vm1 + vm2
```

---

## Tarea 1 — Provisionar infraestructura con plantilla ARM

**Objetivo de la tarea:** Desplegar la red virtual con tres VMs preconfiguradas usando la plantilla oficial del lab. Esto evita configurar manualmente la infraestructura base.

### Método A — Portal web

1. Descarga los archivos `az104-06-vms-template.json` y `az104-06-vms-parameters.json` desde la web oficial del lab

   <img src="capturas/t1-paso-01-descargar-lab06.png" alt="Descargar lab06" width="50%">

2. Portal → **Deploy a custom template**

   <img src="capturas/t1-paso-02-custom-template.png" alt="Custom template" width="50%">

3. **Build your own template in the editor**

   <img src="capturas/t1-paso-03-build-template.png" alt="Build template" width="50%">

4. **Load file** → selecciona `az104-06-vms-template.json`

   <img src="capturas/t1-paso-04-load-file.png" alt="Load file" width="50%">

5. **Edit parameters** → carga `az104-06-vms-parameters.json` → configura contraseña

   <img src="capturas/t1-paso-05-cargar-parameters.png" alt="Cargar parameters" width="50%">

6. Configura el despliegue:
   - **RG:** `az104-rg6` | **Password:** introduce contraseña segura

   <img src="capturas/t1-paso-06-configurar-deployment.png" alt="Configurar deployment" width="50%">

7. **Review + create** → **Create** → espera ~5 minutos

   <img src="capturas/t1-paso-07-recursos-desplegados.png" alt="Recursos desplegados" width="50%">

**Resultado esperado:**
> Una VNet con tres subredes y tres VMs (`az104-06-vm0`, `az104-06-vm1`, `az104-06-vm2`) desplegadas con un servidor web activo en cada una.

---

## Tarea 2 — Configurar Azure Load Balancer

**Objetivo de la tarea:** El Load Balancer opera en capa 4 (TCP/UDP). Distribuye conexiones entrantes entre las VMs del backend pool según un algoritmo round-robin por defecto. Las health probes descartan VMs no disponibles.

### Método A — Portal web

1. Portal → **Load balancers** → **+ Create**

   <img src="capturas/t2-paso-01-buscar-lb.png" alt="Buscar LB" width="50%">
   <img src="capturas/t2-paso-02-crear-lb.png" alt="Crear LB" width="50%">

2. Configura:
   - **RG:** `az104-rg6` | **Name:** `az104-lb` | **SKU:** Standard | **Type:** Public | **Tier:** Regional

   <img src="capturas/t2-paso-03-lb-basics.png" alt="LB basics" width="50%">

3. Pestaña **Frontend IP** → **+ Add**:
   - **Name:** `az104-fe` | **Public IP:** Create new → `az104-lbpip` (Standard, Static)

   <img src="capturas/t2-paso-04-frontend-ip.png" alt="Frontend IP" width="50%">
   <img src="capturas/t2-paso-05-nueva-ip-publica.png" alt="Nueva IP pública" width="50%">
   <img src="capturas/t2-paso-06-frontend-guardado.png" alt="Frontend guardado" width="50%">

4. Pestaña **Backend pools** → **+ Add**:
   - **Name:** `az104-be` | **VNet:** `az104-06-vnet1`
   - Agrega `az104-06-vm0` y `az104-06-vm1`

   <img src="capturas/t2-paso-07-backend-pool-tab.png" alt="Backend pool tab" width="50%">
   <img src="capturas/t2-paso-08-agregar-backend-pool.png" alt="Agregar backend pool" width="50%">

5. **Review + create** → **Create** → **Go to resource**

   <img src="capturas/t2-paso-09-lb-deployed.png" alt="LB deployed" width="50%">
   <img src="capturas/t2-paso-10-lb-overview.png" alt="LB overview" width="50%">
   <img src="capturas/t2-paso-11-lb-go-to-resource.png" alt="LB ir al recurso" width="50%">

6. **Load balancing rules** → **+ Add**:
   - **Name:** `az104-lbrule` | **Frontend:** `az104-fe` | **Backend:** `az104-be`
   - **Protocol:** TCP | **Port:** 80 | **Backend port:** 80
   - **Health probe:** Create new → `az104-hp` (TCP:80, interval 5s)
   - **Session persistence:** None

   <img src="capturas/t2-paso-12-lb-rules.png" alt="LB rules" width="50%">
   <img src="capturas/t2-paso-13-agregar-lb-rule.png" alt="Agregar LB rule" width="50%">
   <img src="capturas/t2-paso-14-health-probe.png" alt="Health probe" width="50%">

7. Copia la IP pública del frontend → abre en el navegador → refresca varias veces

   <img src="capturas/t2-paso-15-copiar-ip-frontend.png" alt="Copiar IP frontend" width="50%">
   <img src="capturas/t2-paso-16-hello-world-vm0.png" alt="Hello World vm0" width="50%">
   <img src="capturas/t2-paso-17-hello-world-vm1.png" alt="Hello World vm1" width="50%">

### Método B — CLI

```powershell
# IP pública
az network public-ip create --name "az104-lbpip" --resource-group "az104-rg6" --sku Standard --allocation-method Static

# Load Balancer
az network lb create --name "az104-lb" --resource-group "az104-rg6" --sku Standard --public-ip-address "az104-lbpip" --frontend-ip-name "az104-fe" --backend-pool-name "az104-be"

# Health probe
az network lb probe create --name "az104-hp" --lb-name "az104-lb" --resource-group "az104-rg6" --protocol tcp --port 80 --interval 5

# Regla de balanceo
az network lb rule create --name "az104-lbrule" --lb-name "az104-lb" --resource-group "az104-rg6" --protocol tcp --frontend-port 80 --backend-port 80 --frontend-ip-name "az104-fe" --backend-pool-name "az104-be" --probe-name "az104-hp"
```

**Resultado esperado:**
> Al acceder a la IP del Load Balancer y refrescar varias veces, el mensaje alterna entre `Hello World from az104-06-vm0` y `Hello World from az104-06-vm1`.

---

## Tarea 3 — Configurar Azure Application Gateway

**Objetivo de la tarea:** El Application Gateway opera en capa 7 (HTTP/HTTPS). Puede tomar decisiones de enrutamiento basadas en la URL, cabeceras o cookies. En este lab enruta `/image/*` a un backend y `/video/*` a otro.

### Método A — Portal web

1. Portal → **Virtual networks** → `az104-06-vnet1` → **Subnets** → **+ Subnet**:
   - **Name:** `subnet-appgw` | **Starting address:** `10.60.3.224/27`

   <img src="capturas/t3-paso-01-buscar-vnets.png" alt="Buscar VNets" width="50%">
   <img src="capturas/t3-paso-02-seleccionar-vnet1.png" alt="Seleccionar vnet1" width="50%">
   <img src="capturas/t3-paso-03-agregar-subnet-appgw.png" alt="Agregar subnet appgw" width="50%">
   <img src="capturas/t3-paso-04-subnet-appgw-config.png" alt="Subnet appgw config" width="50%">

2. Portal → **Application gateways** → **+ Create**

   <img src="capturas/t3-paso-05-buscar-appgw.png" alt="Buscar AppGW" width="50%">
   <img src="capturas/t3-paso-06-crear-appgw.png" alt="Crear AppGW" width="50%">

3. **Basics**:
   - **RG:** `az104-rg6` | **Name:** `az104-appgw` | **Tier:** Standard V2
   - **Autoscaling:** No | **Instances:** 2 | **VNet:** `az104-06-vnet1` | **Subnet:** `subnet-appgw`

   <img src="capturas/t3-paso-07-appgw-basics.png" alt="AppGW basics" width="50%">

4. **Frontends** → IP pública → Create new → `az104-gwpip`

   <img src="capturas/t3-paso-08-frontend-ip.png" alt="Frontend IP" width="50%">

5. **Backends** → Agrega tres pools:
   - `az104-appgwbe`: vm1 + vm2
   - `az104-imagebe`: vm1 (nic1)
   - `az104-videobe`: vm2 (nic2)

   <img src="capturas/t3-paso-09-backend-general.png" alt="Backend general" width="50%">
   <img src="capturas/t3-paso-10-backend-imagenes.png" alt="Backend imagenes" width="50%">
   <img src="capturas/t3-paso-11-backend-videos.png" alt="Backend videos" width="50%">

6. **Configuration** → **+ Add routing rule**:
   - **Rule name:** `az104-gwrule` | **Priority:** 10
   - **Listener:** `az104-listener` | HTTP:80

   <img src="capturas/t3-paso-12-configuration-tab.png" alt="Configuration tab" width="50%">
   <img src="capturas/t3-paso-13-agregar-routing-rule.png" alt="Agregar routing rule" width="50%">

7. **Backend targets** → `az104-appgwbe` → **HTTP settings:** Create new `az104-http`

   <img src="capturas/t3-paso-14-http-settings.png" alt="HTTP settings" width="50%">

8. **Path-based routing** → agrega:
   - `/image/*` → `az104-imagebe`
   - `/video/*` → `az104-videobe`

   <img src="capturas/t3-paso-15-path-based-image.png" alt="Path based image" width="50%">
   <img src="capturas/t3-paso-16-path-based-video.png" alt="Path based video" width="50%">
   <img src="capturas/t3-paso-17-path-rules-configuradas.png" alt="Path rules configuradas" width="50%">

9. **Review + create** → **Create** → espera 5-10 minutos

   <img src="capturas/t3-paso-18-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t3-paso-19-deployment-completado.png" alt="Deployment completado" width="50%">

10. Verifica **Backend health** → ambos backends deben estar **Healthy**

    <img src="capturas/t3-paso-20-backend-health.png" alt="Backend health" width="50%">

11. Prueba en el navegador:
    - `http://<ip-appgw>/image/` → vm1
    - `http://<ip-appgw>/video/` → vm2

    <img src="capturas/t3-paso-21-test-image-path.png" alt="Test image path" width="50%">
    <img src="capturas/t3-paso-22-test-video-path.png" alt="Test video path" width="50%">

**Resultado esperado:**
> Las rutas `/image/` y `/video/` redirigen a servidores distintos. El Application Gateway demuestra routing basado en contenido de la URL.

---

## ⚠️ Errores comunes

### Error 1 — Backend health muestra Unhealthy

**Síntoma:** El Application Gateway muestra los backends como Unhealthy.

**Causa:** El servidor web en las VMs no está corriendo en el puerto 80, o el NSG bloquea el probe del App Gateway.

**Solución:** Verifica que el NSG de las VMs permite tráfico desde `GatewayManager` (tag de servicio) y desde la subred `subnet-appgw`.

---

### Error 2 — El LB no alterna entre VMs

**Síntoma:** La respuesta siempre viene de la misma VM.

**Causa:** La **Session persistence** está configurada como `Client IP` en lugar de `None`. Con persistencia de sesión, las peticiones del mismo cliente van siempre al mismo backend.

**Solución:** Edita la regla de LB → **Session persistence:** None. También puede ser necesario usar una ventana de incógnito para evitar el caché del browser.

---

## 🧹 Limpieza de recursos

> ⚠️ El Application Gateway V2 cobra ~$0.22/hora incluso sin tráfico. Elimina el RG inmediatamente.

### Método A — Portal

`az104-rg6` → **Eliminar grupo de recursos**

<img src="capturas/limpieza-eliminar-rg.png" alt="Limpieza" width="50%">
<img src="capturas/limpieza-confirmacion.png" alt="Limpieza confirmación" width="50%">

### Método B — CLI

```powershell
az group delete --name "az104-rg6" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| 3× VM Standard_D2s_v3 | ~1 hora | ~$0.56 |
| Load Balancer Standard | ~1 hora | ~$0.025 |
| Application Gateway V2 (2 instancias) | ~1 hora | ~$0.22 |
| **Total lab** | | **~$0.85** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Balanceo capa 4 | Load Balancer Standard | NLB | TCP/UDP Load Balancer | Network Load Balancer |
| Balanceo capa 7 | Application Gateway | ALB | HTTP(S) Load Balancer | Flexible Load Balancer |
| Path-based routing | Sí (App Gateway) | Sí (ALB) | Sí | Sí |
| WAF integrado | Sí (App Gateway WAF v2) | Sí (AWS WAF con ALB) | Sí (Cloud Armor) | Sí |
| Costo LB/hora | ~$0.025 | ~$0.018 | ~$0.025 | ~$0.013 |
| Costo App GW/hora | ~$0.22 | ~$0.028 (ALB) | ~$0.025 | ~$0.016 |

**¿Cuándo usar Load Balancer vs Application Gateway?**
> Load Balancer para tráfico TCP/UDP sin necesidad de inspección de contenido (bases de datos, servicios internos). Application Gateway cuando necesitas routing por URL, terminación SSL, WAF o afinidad de sesión por cookie. En producción se usan juntos: LB en la capa interna, App Gateway en la capa pública.

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- Load Balancer (capa 4) es simple y de alto rendimiento pero no entiende el contenido HTTP
- Application Gateway (capa 7) puede enrutar por URL, añadir WAF y terminar SSL — tiene mayor latencia y costo
- Las health probes son fundamentales: sin ellas el balanceador envía tráfico a backends caídos
- El Application Gateway necesita una subred dedicada `/27` o mayor

**Cómo se usa en labs futuros:**
> Los conceptos de backend pools y health probes se aplican en el **Lab-08 VMs** con Virtual Machine Scale Sets y Load Balancer automático.

**Siguiente lab recomendado:**
👉 [Lab 07 — Azure Storage](../../Lab-07-Storage/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
