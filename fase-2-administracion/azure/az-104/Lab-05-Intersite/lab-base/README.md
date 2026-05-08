# Intersite Connectivity — Lab 05

> Fase: F2 | Cert: AZ-104 | Lab: 05 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 05
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.50/hora (2× VM Standard_D2s_v3 + peering)
**Región usada:** East US
**Duración estimada:** 50 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Crear CoreServicesVM en su VNet](#tarea-1)
- [Tarea 2 — Crear ManufacturingVM en su VNet](#tarea-2)
- [Tarea 3 — Verificar conectividad con Network Watcher](#tarea-3)
- [Tarea 4 — Configurar VNet Peering](#tarea-4)
- [Tarea 5 — Probar conectividad con PowerShell](#tarea-5)
- [Tarea 6 — Crear ruta personalizada UDR](#tarea-6)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Demostrar que dos VNets sin peering no tienen conectividad entre sí
- [ ] Configurar VNet Peering bidireccional entre dos redes virtuales
- [ ] Verificar conectividad con Network Watcher y Test-NetConnection
- [ ] Crear rutas personalizadas (UDR) para dirigir tráfico a través de un NVA

**Habilidades del examen que cubre este lab:**
> **Configure and manage virtual networking (25–30%)**
> — Configure VNet peering
> — Configure user-defined routes
> — Diagnose and resolve connectivity issues using Network Watcher

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 04 — Virtual Networking *(conocimiento de VNets y subredes)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- Azure CLI instalada

**Conocimientos previos:**
- VNet Peering es no transitivo: A↔B y B↔C no implica A↔C
- Test-NetConnection equivale a `telnet` en Windows — verifica conectividad TCP a un puerto específico

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| VM Standard_D2s_v3 Windows | CoreServicesVM | ~$0.188/hora |
| VM Standard_D2s_v3 Windows | ManufacturingVM | ~$0.188/hora |
| VNet Peering | Conectividad entre VNets | ~$0.01/GB transferido |
| Network Watcher | Diagnóstico de conectividad | $0.00 |
| Route Table | Tabla de rutas UDR | $0.00 |

---

## 🏗️ Arquitectura del lab

```
East US
│
├── CoreServicesVnet (10.0.0.0/16)
│   ├── Core (10.0.0.0/24)
│   │   └── CoreServicesVM
│   └── perimeter (10.0.1.0/24)  ← Tarea 6
│       └── [NVA futuro: 10.0.1.7]
│
├── ManufacturingVnet (172.16.0.0/16)
│   └── Manufacturing (172.16.0.0/24)
│       └── ManufacturingVM
│
├── Peering: CoreServicesVnet ↔ ManufacturingVnet  ← Tarea 4
│
└── Route Table: rt-CoreServices
    └── Ruta: PerimetertoCore → 10.0.0.0/16 via 10.0.1.7
```

---

## Tarea 1 — Crear CoreServicesVM en su VNet

**Objetivo de la tarea:** Crear la primera VM junto con su red virtual. Azure permite crear la VNet en el mismo flujo de creación de la VM.

### Método A — Portal web

1. Portal → **Virtual machines** → **+ Create** → **Azure virtual machine**

   <img src="capturas/t1-paso-01-buscar-vm.png" alt="Buscar VM" width="50%">
   <img src="capturas/t1-paso-02-crear-vm.png" alt="Crear VM" width="50%">

2. Configura la pestaña **Basics**:
   - **RG:** `az104-rg5` | **Name:** `CoreServicesVM` | **Region:** East US
   - **Security type:** Standard | **Image:** Windows Server 2025 Datacenter x64 Gen2
   - **Size:** Standard_D2s_v3 | **Username:** `localadmin` | **Inbound ports:** None

   <img src="capturas/t1-paso-03-basics-coreservicesvm.png" alt="Basics CoreServicesVM" width="50%">
   <img src="capturas/t1-paso-04-basics-continuacion.png" alt="Basics continuación" width="50%">

3. Pestaña **Networking** → **Virtual network: Create new** → configura:
   - **Name:** `CoreServicesVnet` | **Address range:** `10.0.0.0/16`
   - **Subnet:** `Core` | `10.0.0.0/24`

   <img src="capturas/t1-paso-05-crear-coreservicesvnet.png" alt="Crear CoreServicesVnet" width="50%">

4. Pestaña **Monitoring** → **Boot diagnostics:** Disabled → **Review + create** → **Create**

   <img src="capturas/t1-paso-06-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t1-paso-07-deployment-iniciado.png" alt="Deployment iniciado" width="50%">
   <img src="capturas/t1-paso-08-deployment-completado.png" alt="Deployment completado" width="50%">

> ⏱️ No es necesario esperar — continúa con la Tarea 2 mientras se despliega.

**Resultado esperado:**
> `CoreServicesVM` creándose en `CoreServicesVnet (10.0.0.0/16)` subred `Core (10.0.0.0/24)`.

---

## Tarea 2 — Crear ManufacturingVM en su VNet

**Objetivo de la tarea:** Crear la segunda VM en una VNet diferente y en un rango de IPs distinto para evitar solapamiento.

### Método A — Portal web

1. Portal → **Virtual machines** → **+ Create**

   <img src="capturas/t2-paso-01-nueva-vm.png" alt="Nueva VM" width="50%">

2. Configura:
   - **RG:** `az104-rg5` | **Name:** `ManufacturingVM` | **Region:** East US
   - **Image:** Windows Server 2025 | **Size:** Standard_D2s_v3 | **Inbound ports:** None

   <img src="capturas/t2-paso-02-basics-manufacturingvm.png" alt="Basics ManufacturingVM" width="50%">
   <img src="capturas/t2-paso-03-basics-continuacion.png" alt="Basics continuación" width="50%">

3. **Networking** → **Create new VNet**:
   - **Name:** `ManufacturingVnet` | **Address range:** `172.16.0.0/16`
   - **Subnet:** `Manufacturing` | `172.16.0.0/24`

   <img src="capturas/t2-paso-04-crear-manufacturingvnet.png" alt="Crear ManufacturingVnet" width="50%">

4. **Monitoring** → Disabled → **Review + create** → **Create**

   <img src="capturas/t2-paso-05-monitoring-disabled.png" alt="Monitoring disabled" width="50%">
   <img src="capturas/t2-paso-06-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t2-paso-07-deployment-completado.png" alt="Deployment completado" width="50%">

**Resultado esperado:**
> Dos VMs en dos VNets distintas, ambas en East US, sin conectividad entre sí todavía.

---

## Tarea 3 — Verificar conectividad con Network Watcher

**Objetivo de la tarea:** Confirmar que sin peering las VMs no pueden comunicarse. Network Watcher diagnostica el camino de red entre dos recursos.

### Método A — Portal web

1. Portal → **Network Watcher** → **Connection troubleshoot**

   <img src="capturas/t3-paso-01-buscar-network-watcher.png" alt="Buscar Network Watcher" width="50%">
   <img src="capturas/t3-paso-02-connection-troubleshoot.png" alt="Connection troubleshoot" width="50%">

2. Configura el test:
   - **Source:** `CoreServicesVM`
   - **Destination:** `ManufacturingVM`
   - **Protocol:** TCP | **Port:** 3389

   <img src="capturas/t3-paso-03-configurar-test.png" alt="Configurar test" width="50%">

3. **Ejecutar pruebas de diagnóstico** → resultado esperado: **INACCESIBLE**

   <img src="capturas/t3-paso-04-ejecutar-diagnostico.png" alt="Ejecutar diagnóstico" width="50%">

> El resultado INACCESIBLE es correcto — las VNets están aisladas sin peering.

**Resultado esperado:**
> Network Watcher confirma que no hay conectividad entre las dos VMs.

---

## Tarea 4 — Configurar VNet Peering

**Objetivo de la tarea:** Crear peering bidireccional entre las dos VNets. En Azure, un peering crea automáticamente dos vínculos: uno en cada dirección.

### Método A — Portal web

1. Portal → **CoreServicesVnet** → **Emparejamientos** → **+ Agregar**

   <img src="capturas/t4-paso-01-coreservicesvnet.png" alt="CoreServicesVnet" width="50%">
   <img src="capturas/t4-paso-02-ir-a-vnet.png" alt="Ir a VNet" width="50%">
   <img src="capturas/t4-paso-03-menu-peerings.png" alt="Menú peerings" width="50%">

2. Configura el peering:
   - **Nombre del vínculo:** `ManufacturingVnet-to-CoreServicesVnet`
   - **Red virtual:** `ManufacturingVnet`
   - **Peering link name (remoto):** `CoreServicesVnet-to-ManufacturingVnet`
   - Habilita tráfico en ambas direcciones

   <img src="capturas/t4-paso-04-agregar-peering.png" alt="Agregar peering" width="50%">
   <img src="capturas/t4-paso-05-configurar-peering.png" alt="Configurar peering" width="50%">

3. Verifica el estado: **Connected**

   <img src="capturas/t4-paso-06-peering-coreservices-creado.png" alt="Peering CoreServices creado" width="50%">
   <img src="capturas/t4-paso-07-peering-manufacturing-vista.png" alt="Peering Manufacturing vista" width="50%">
   <img src="capturas/t4-paso-08-peering-connected.png" alt="Peering connected" width="50%">

### Método B — CLI

```powershell
$coreVnetId = az network vnet show --name "CoreServicesVnet" --resource-group "az104-rg5" --query id --output tsv
$mfgVnetId  = az network vnet show --name "ManufacturingVnet" --resource-group "az104-rg5" --query id --output tsv

# Peering Core → Manufacturing
az network vnet peering create `
  --name "CoreToManufacturing" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg5" `
  --remote-vnet $mfgVnetId `
  --allow-vnet-access

# Peering Manufacturing → Core
az network vnet peering create `
  --name "ManufacturingToCore" `
  --vnet-name "ManufacturingVnet" `
  --resource-group "az104-rg5" `
  --remote-vnet $coreVnetId `
  --allow-vnet-access
```

**Resultado esperado:**
> Ambos peerings muestran estado **Connected**. El tráfico entre las VNets ahora fluye por la red troncal de Microsoft.

---

## Tarea 5 — Probar conectividad con PowerShell

**Objetivo de la tarea:** Verificar que el peering funciona ejecutando `Test-NetConnection` desde ManufacturingVM hacia la IP privada de CoreServicesVM.

### Método A — Portal web

1. Anota la IP privada de `CoreServicesVM` desde su Overview → sección Redes

   <img src="capturas/t5-paso-01-ip-privada-coreservicesvm.png" alt="IP privada CoreServicesVM" width="50%">

2. Navega a `ManufacturingVM`

   <img src="capturas/t5-paso-02-ir-manufacturingvm.png" alt="Ir ManufacturingVM" width="50%">
   <img src="capturas/t5-paso-03-manufacturing-overview.png" alt="Manufacturing overview" width="50%">

3. **Operaciones** → **Ejecutar comando** → **RunPowerShellScript**

   <img src="capturas/t5-paso-04-run-command.png" alt="Run command" width="50%">
   <img src="capturas/t5-paso-05-runpowershellscript.png" alt="RunPowerShellScript" width="50%">

4. Ejecuta el comando (reemplaza la IP):

```powershell
Test-NetConnection <IP-privada-de-CoreServicesVM> -port 3389
```

   <img src="capturas/t5-paso-06-test-netconnection-cmd.png" alt="Test-NetConnection cmd" width="50%">
   <img src="capturas/t5-paso-07-ejecutar-script.png" alt="Ejecutar script" width="50%">

5. Resultado: `TcpTestSucceeded: True`

   <img src="capturas/t5-paso-08-resultado-conectado.png" alt="Resultado conectado" width="50%">

**Resultado esperado:**
> `TcpTestSucceeded: True` — el peering permite comunicación directa entre las VMs usando IPs privadas.

---

## Tarea 6 — Crear ruta personalizada UDR

**Objetivo de la tarea:** Crear una tabla de rutas que dirija el tráfico desde la subred `perimeter` hacia una IP de NVA (Network Virtual Appliance) futuro. Las UDR sobreescriben el enrutamiento automático de Azure.

### Método A — Portal web

1. En `CoreServicesVnet` → **Subredes** → **+ Subred**:
   - **Name:** `perimeter` | **Starting address:** `10.0.1.0/24`

   <img src="capturas/t6-paso-01-coreservicesvnet-subnets.png" alt="CoreServicesVnet subnets" width="50%">
   <img src="capturas/t6-paso-02-ver-subnets.png" alt="Ver subnets" width="50%">
   <img src="capturas/t6-paso-03-crear-subnet-perimeter.png" alt="Crear subnet perimeter" width="50%">

2. Portal → **Route tables** → **+ Crear**:
   - **RG:** `az104-rg5` | **Region:** East US | **Name:** `rt-CoreServices`
   - **Propagar rutas de puertas de enlace:** No

   <img src="capturas/t6-paso-04-buscar-route-tables.png" alt="Buscar route tables" width="50%">
   <img src="capturas/t6-paso-05-crear-route-table.png" alt="Crear route table" width="50%">
   <img src="capturas/t6-paso-06-route-table-basics.png" alt="Route table basics" width="50%">
   <img src="capturas/t6-paso-07-route-table-creada.png" alt="Route table creada" width="50%">

3. Selecciona `rt-CoreServices` → **Rutas** → **+ Agregar**:
   - **Route name:** `PerimetertoCore`
   - **Destination:** `10.0.0.0/16`
   - **Next hop type:** Virtual appliance
   - **Next hop address:** `10.0.1.7`

   <img src="capturas/t6-paso-08-ir-rt-coreservices.png" alt="Ir rt-CoreServices" width="50%">
   <img src="capturas/t6-paso-09-agregar-ruta.png" alt="Agregar ruta" width="50%">
   <img src="capturas/t6-paso-10-configurar-ruta-nva.png" alt="Configurar ruta NVA" width="50%">
   <img src="capturas/t6-paso-11-ruta-guardada.png" alt="Ruta guardada" width="50%">

4. **Subnets** → **+ Associate** → `CoreServicesVnet / perimeter`

   <img src="capturas/t6-paso-12-asociar-subnet.png" alt="Asociar subnet" width="50%">
   <img src="capturas/t6-paso-13-subnet-perimeter-asociada.png" alt="Subnet perimeter asociada" width="50%">

### Método B — CLI

```powershell
# Crear subred perimeter
az network vnet subnet create `
  --name "perimeter" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg5" `
  --address-prefix "10.0.1.0/24"

# Crear tabla de rutas
az network route-table create `
  --name "rt-CoreServices" `
  --resource-group "az104-rg5" `
  --location "eastus" `
  --disable-bgp-route-propagation true

# Agregar ruta hacia NVA
az network route-table route create `
  --name "PerimetertoCore" `
  --route-table-name "rt-CoreServices" `
  --resource-group "az104-rg5" `
  --address-prefix "10.0.0.0/16" `
  --next-hop-type VirtualAppliance `
  --next-hop-ip-address "10.0.1.7"

# Asociar a subred perimeter
az network vnet subnet update `
  --name "perimeter" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg5" `
  --route-table "rt-CoreServices"
```

**Resultado esperado:**
> La subred `perimeter` tiene asociada la tabla de rutas `rt-CoreServices`. Todo el tráfico con destino `10.0.0.0/16` pasa por `10.0.1.7` (NVA futuro).

---

## ⚠️ Errores comunes

### Error 1 — Peering falla con `RemoteVnetLacksPermission`

**Síntoma:** Error al crear el peering desde una VNet.

**Causa:** No tienes permisos de **Network Contributor** en la VNet remota (si está en otra suscripción o RG).

**Solución:** Asegúrate de tener rol Contributor en ambas VNets o usa el parámetro `--remote-vnet` con el ID completo.

---

### Error 2 — `Test-NetConnection` devuelve False después del peering

**Síntoma:** El peering está Connected pero la conectividad no funciona.

**Causa:** El NSG de alguna de las VMs bloquea el puerto 3389 (RDP). Por defecto las VMs de este lab tienen "No inbound ports" seleccionado.

**Solución:**
```powershell
# Agregar regla NSG para permitir RDP entre VNets
az network nsg rule create `
  --name "AllowRDPFromVnet" `
  --nsg-name "<nombre-nsg>" `
  --resource-group "az104-rg5" `
  --priority 200 `
  --source-address-prefixes "VirtualNetwork" `
  --destination-port-ranges 3389 `
  --access Allow
```

---

## 🧹 Limpieza de recursos

> ⚠️ Las VMs generan costo mientras están encendidas (~$0.38/hora). Elimina el RG al terminar.

### Método A — Portal

`az104-rg5` → **Eliminar grupo de recursos**

### Método B — CLI

```powershell
az group delete --name "az104-rg5" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| 2× VM Standard_D2s_v3 | ~1 hora | ~$0.38 |
| VNet Peering (tráfico mínimo) | ~1 hora | ~$0.01 |
| **Total lab** | | **~$0.50** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Conectividad entre redes | VNet Peering | VPC Peering | VPC Peering | Local/Remote Peering |
| Rutas personalizadas | UDR (Route Table) | Route Table | Cloud Router | Route Table |
| Diagnóstico de red | Network Watcher | VPC Reachability Analyzer | Network Intelligence Center | Network Path Analyzer |
| Costo peering/GB | ~$0.01 | ~$0.01 | ~$0.01 | ~$0.008 |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- VNet Peering en Azure no es transitivo — cada par de VNets necesita su propio peering
- Network Watcher es la herramienta de diagnóstico nativa: úsala antes de añadir reglas NSG complejas
- Las UDR permiten forzar tráfico a través de NVAs para inspección de seguridad — patrón habitual en arquitecturas hub-and-spoke

**Cómo se usa en labs futuros:**
> La conectividad entre VNets establecida aquí es la base para el **Lab-06 Traffic Manager**, donde el tráfico entre las VMs se gestiona con Load Balancer y Application Gateway.

**Siguiente lab recomendado:**
👉 [Lab 06 — Traffic Manager](../../Lab-06-Traffic-Manager/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
