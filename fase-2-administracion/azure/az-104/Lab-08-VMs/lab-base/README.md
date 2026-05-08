# Virtual Machines — Lab 08

> Fase: F2 | Cert: AZ-104 | Lab: 08 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 08
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.65 USD/hora (2 VMs + VMSS 2 instancias + LB)
**Región usada:** East US
**Duración estimada:** 60 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — VMs resilientes por zonas de disponibilidad](#tarea-1)
- [Tarea 2 — Escalado de cómputo y almacenamiento](#tarea-2)
- [Tarea 3 — Crear Virtual Machine Scale Set](#tarea-3)
- [Tarea 4 — Configurar autoscaling en VMSS](#tarea-4)
- [Tarea 5 — Crear VM con PowerShell](#tarea-5)
- [Tarea 6 — Crear VM con CLI](#tarea-6)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Desplegar VMs en múltiples zonas de disponibilidad para SLA del 99.99%
- [ ] Escalar verticalmente una VM (cambio de SKU) y modificar discos administrados
- [ ] Crear un Virtual Machine Scale Set con Load Balancer y NSG
- [ ] Configurar autoscaling basado en métricas de CPU
- [ ] Desplegar VMs desde PowerShell y CLI

**Habilidades del examen que cubre este lab:**
> **Deploy and manage Azure compute resources (20–25%)**
> — Configure virtual machines
> — Configure virtual machine availability
> — Configure Virtual Machine Scale Sets

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 04 — Virtual Networking
- [ ] Lab 06 — Traffic Management *(conceptos de Load Balancer)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- Azure CLI y Azure PowerShell instalados

**Conocimientos previos:**
- Diferencia entre escalado vertical (más CPU/RAM en la misma VM) y horizontal (más instancias)
- Availability Zones: datacenters físicamente separados dentro de una región — protegen contra fallos de datacenter

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| 2× VM Standard_D2s_v3 Windows | VMs en zonas distintas | ~$0.42/hora |
| VM Scale Set (2 instancias) | Escalado horizontal automático | ~$0.21/hora |
| Load Balancer Standard | Balanceo del VMSS | ~$0.025/hora |
| Managed Disk Standard HDD | Disco adicional de práctica | ~$0.002/hora |

---

## 🏗️ Arquitectura del lab

```
Resource Group: az104-rg8
│
├── Tarea 1 — VMs por zonas
│   ├── az104-vm1 → Availability Zone 1
│   └── az104-vm2 → Availability Zone 2
│         (SLA 99.99% con 2 zonas distintas)
│
├── Tarea 3 — VMSS
│   └── vmss1 → Zones 1, 2, 3
│       ├── vmss-vnet (10.82.0.0/20)
│       │   └── subnet0 (10.82.0.0/24)
│       ├── NSG: vmss1-nsg (allow HTTP:80)
│       └── Load Balancer: vmss-lb (IP pública)
│
└── Tarea 5/6 — VMs adicionales
    ├── myPSVM  (creada con PowerShell)
    └── myCLIVM (creada con CLI, Ubuntu)
```

---

## Tarea 1 — VMs resilientes por zonas de disponibilidad

**Objetivo de la tarea:** Desplegar dos VMs en zonas de disponibilidad distintas. Con al menos dos VMs en zonas diferentes, el SLA sube al 99.99% — si un datacenter completo falla, la otra VM sigue operativa.

### Método A — Portal web

1. Portal → **Virtual machines** → **+ Create** → **Azure virtual machine**

   <img src="capturas/t1-paso-01-buscar-vms.png" alt="Buscar VMs" width="50%">
   <img src="capturas/t1-paso-02-crear-vm-menu.png" alt="Crear VM menú" width="50%">

2. En **Availability zone** → marca **Zone 1** y **Zone 2** → aparece opción *Edit names*

   <img src="capturas/t1-paso-03-seleccionar-zonas.png" alt="Seleccionar zonas" width="50%">

3. Edita los nombres: `az104-vm1` (Zone 1) y `az104-vm2` (Zone 2)

   <img src="capturas/t1-paso-04-editar-nombres-vm.png" alt="Editar nombres VM" width="50%">

4. Configura el resto:
   - **RG:** `az104-rg8` | **Region:** East US | **Security type:** Standard
   - **Image:** Windows Server 2025 Datacenter x64 Gen2 | **Size:** Standard_D2s_v3
   - **Username:** `localadmin` | **Inbound ports:** None

   <img src="capturas/t1-paso-05-basics-completado.png" alt="Basics completado" width="50%">

5. **Disks** → **OS disk type:** Premium SSD

   <img src="capturas/t1-paso-06-disks-premium-ssd.png" alt="Disks Premium SSD" width="50%">

6. **Networking** → **Load balancing:** None | **Delete public IP when VM deleted:** checked

   <img src="capturas/t1-paso-07-networking-config.png" alt="Networking config" width="50%">

7. **Monitoring** → Boot diagnostics: Disable → **Review + create** → **Create**

   <img src="capturas/t1-paso-08-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t1-paso-09-deployment-completado.png" alt="Deployment completado" width="50%">

### Método B — CLI

```powershell
az group create --name "az104-rg8" --location "eastus"

# VM en Zona 1
az vm create --name "az104-vm1" --resource-group "az104-rg8" --location "eastus" `
  --image "Win2022Datacenter" --size "Standard_D2s_v3" --zone 1 `
  --admin-username "localadmin" --admin-password "P@ssword123456!" `
  --os-disk-sku "Premium_LRS" --public-ip-address ""

# VM en Zona 2
az vm create --name "az104-vm2" --resource-group "az104-rg8" --location "eastus" `
  --image "Win2022Datacenter" --size "Standard_D2s_v3" --zone 2 `
  --admin-username "localadmin" --admin-password "P@ssword123456!" `
  --os-disk-sku "Premium_LRS" --public-ip-address ""
```

**Resultado esperado:**
> Dos VMs en zonas distintas, cada una con su propia NIC e IP. En caso de fallo de zona, la otra VM sigue respondiendo.

---

## Tarea 2 — Escalado de cómputo y almacenamiento

**Objetivo de la tarea:** Cambiar el SKU de `az104-vm1` (escalado vertical) y practicar con discos: crear, desvincular, cambiar tipo y reconectar.

### Método A — Portal web

1. `az104-vm1` → **Availability + scale → Size** → selecciona **D2ds_v4** → **Resize**

   <img src="capturas/t2-paso-01-cambiar-size-vm.png" alt="Cambiar size VM" width="50%">
   <img src="capturas/t2-paso-02-seleccionar-d2ds-v4.png" alt="Seleccionar D2ds v4" width="50%">

2. **Settings → Disks** → **+ Create and attach a new disk**:
   - **Name:** `vm1-disk1` | **Type:** Standard HDD | **Size:** 32 GiB → **Apply**

   <img src="capturas/t2-paso-03-disks-menu.png" alt="Disks menú" width="50%">
   <img src="capturas/t2-paso-04-crear-disco-vm1disk1.png" alt="Crear disco vm1-disk1" width="50%">

3. **Detach** el disco → **Apply**

   <img src="capturas/t2-paso-05-detach-disco.png" alt="Detach disco" width="50%">

4. Portal → **Disks** → selecciona `vm1-disk1`

   <img src="capturas/t2-paso-06-buscar-discos.png" alt="Buscar discos" width="50%">
   <img src="capturas/t2-paso-07-disco-overview.png" alt="Disco overview" width="50%">

5. **Settings → Size + performance** → cambia a **Standard SSD** → **Save**

   <img src="capturas/t2-paso-08-size-performance.png" alt="Size performance" width="50%">
   <img src="capturas/t2-paso-09-cambiar-a-standard-ssd.png" alt="Cambiar a Standard SSD" width="50%">
   <img src="capturas/t2-paso-10-disco-ssd-guardado.png" alt="Disco SSD guardado" width="50%">

6. Vuelve a `az104-vm1` → **Disks** → **Attach existing disks** → selecciona `vm1-disk1` → **Apply**

   <img src="capturas/t2-paso-11-disks-vm1.png" alt="Disks vm1" width="50%">
   <img src="capturas/t2-paso-12-attach-existing-disk.png" alt="Attach existing disk" width="50%">

**Resultado esperado:**
> `az104-vm1` con SKU D2ds_v4 y disco `vm1-disk1` de tipo Standard SSD reconectado.

---

## Tarea 3 — Crear Virtual Machine Scale Set

**Objetivo de la tarea:** Los VMSS crean y eliminan instancias automáticamente según la carga. A diferencia de VMs individuales, no administras instancias — defines el modelo y Azure gestiona el pool.

### Método A — Portal web

1. Portal → **Virtual machine scale sets** → **+ Create**

   <img src="capturas/t3-paso-01-buscar-vmss.png" alt="Buscar VMSS" width="50%">
   <img src="capturas/t3-paso-02-crear-vmss.png" alt="Crear VMSS" width="50%">

2. Configura **Basics**:
   - **RG:** `az104-rg8` | **Name:** `vmss1` | **Region:** East US
   - **Zones:** 1, 2, 3 | **Orchestration:** Uniform | **Image:** Windows Server 2025

3. **Networking** → **Edit virtual network link**:
   - **Name:** `vmss-vnet` | **Address range:** `10.82.0.0/20`
   - **Subnet:** `subnet0` | `10.82.0.0/24`

   <img src="capturas/t3-paso-03-vmss-networking.png" alt="VMSS networking" width="50%">

4. **Edit network interface** → **NIC NSG: Advanced** → **Create new**:
   - **Name:** `vmss1-nsg` | Agrega regla: Allow HTTP:80

   <img src="capturas/t3-paso-04-edit-nic.png" alt="Edit NIC" width="50%">
   <img src="capturas/t3-paso-05-crear-nsg-vmss.png" alt="Crear NSG VMSS" width="50%">

5. Habilita IP pública → **Load balancing: Azure load balancer** → **Create new:** `vmss-lb`

   <img src="capturas/t3-paso-06-crear-load-balancer.png" alt="Crear Load Balancer" width="50%">

6. **Management** → Boot diagnostics: Disable → **Review + create** → **Create**

   <img src="capturas/t3-paso-07-vmss-deployed.png" alt="VMSS deployed" width="50%">
   <img src="capturas/t3-paso-08-vmss-overview.png" alt="VMSS overview" width="50%">

**Resultado esperado:**
> VMSS `vmss1` con 2 instancias iniciales distribuidas en 3 zonas, con NSG y Load Balancer configurados.

---

## Tarea 4 — Configurar autoscaling en VMSS

**Objetivo de la tarea:** Definir reglas para que el VMSS escale automáticamente: aumenta instancias cuando CPU > 70% y las reduce cuando CPU < 30%.

### Método A — Portal web

1. `vmss1` → **Availability + Scale → Scaling** → **Custom autoscale**

   <img src="capturas/t4-paso-01-vmss-overview.png" alt="VMSS overview" width="50%">
   <img src="capturas/t4-paso-02-custom-autoscale.png" alt="Custom autoscale" width="50%">

2. **Scale based on metric** → **+ Add a rule** (Scale OUT):
   - Metric: CPU% | Operator: Greater than | Threshold: 70 | Duration: 10 min
   - Action: Increase percent by 50% | Cooldown: 5 min

   <img src="capturas/t4-paso-03-scale-based-metric.png" alt="Scale based metric" width="50%">
   <img src="capturas/t4-paso-04-regla-scale-out.png" alt="Regla scale out" width="50%">

3. **+ Add a rule** (Scale IN):
   - Metric: CPU% | Operator: Less than | Threshold: 30
   - Action: Decrease percentage by 50%

   <img src="capturas/t4-paso-05-regla-scale-in.png" alt="Regla scale in" width="50%">

4. Configura los límites de instancias:
   - **Minimum:** 2 | **Maximum:** 10 | **Default:** 2 → **Save**

   <img src="capturas/t4-paso-06-limites-instancias.png" alt="Límites instancias" width="50%">

5. **Instances** → monitorea las instancias activas

   <img src="capturas/t4-paso-07-instancias-activas.png" alt="Instancias activas" width="50%">

**Resultado esperado:**
> VMSS configurado con autoscaling: escala de 2 a 10 instancias según CPU, con cooldown de 5 minutos entre ajustes.

---

## Tarea 5 — Crear VM con PowerShell

**Objetivo de la tarea:** Automatizar la creación de VMs desde PowerShell. En producción, los scripts de PowerShell se integran en pipelines CI/CD para despliegues consistentes.

### Método A — Cloud Shell (PowerShell)

1. Abre Cloud Shell → selecciona **PowerShell**

2. Crea la VM (te pedirá credenciales interactivamente):

```powershell
New-AzVm `
  -ResourceGroupName "az104-rg8" `
  -Name "myPSVM" `
  -Location "East US" `
  -Image "Win2019Datacenter" `
  -Zone "1" `
  -Size "Standard_D2s_v3" `
  -Credential (Get-Credential)
```

   <img src="capturas/t5-paso-01-cloudshell-new-azvm.png" alt="CloudShell New-AzVM" width="50%">
   <img src="capturas/t5-paso-02-solicitar-credenciales.png" alt="Solicitar credenciales" width="50%">
   <img src="capturas/t5-paso-03-vm-creando.png" alt="VM creando" width="50%">
   <img src="capturas/t5-paso-04-vm-creada.png" alt="VM creada" width="50%">

3. Verifica el estado:

```powershell
Get-AzVM -ResourceGroupName "az104-rg8" -Status
```

   <img src="capturas/t5-paso-05-get-azvm-status.png" alt="Get-AzVM status" width="50%">

4. Desasigna la VM para parar el cómputo:

```powershell
Stop-AzVM -ResourceGroupName "az104-rg8" -Name "myPSVM"
```

   <img src="capturas/t5-paso-06-stop-azvm.png" alt="Stop-AzVM" width="50%">
   <img src="capturas/t5-paso-07-vm-deallocated.png" alt="VM deallocated" width="50%">

**Resultado esperado:**
> `myPSVM` en estado `Deallocated` — deja de generar costo de cómputo (el disco sigue cobrando).

---

## Tarea 6 — Crear VM con CLI

**Objetivo de la tarea:** Crear una VM Linux desde CLI. Contrasta con PowerShell: misma capacidad, sintaxis distinta.

### Método A — Cloud Shell (Bash)

1. Cambia a **Bash** en Cloud Shell

   <img src="capturas/t6-paso-01-cambiar-bash.png" alt="Cambiar Bash" width="50%">

2. Crea la VM Ubuntu:

```bash
az vm create \
  --name "myCLIVM" \
  --resource-group "az104-rg8" \
  --image "Canonical:0001-com-ubuntu-server-jammy:22_04-lts:latest" \
  --admin-username "localadmin" \
  --generate-ssh-keys \
  --size "Standard_D2s_v3" \
  --location "eastus"
```

   <img src="capturas/t6-paso-02-az-vm-create.png" alt="az vm create" width="50%">

3. Verifica el estado:

```bash
az vm show --name "myCLIVM" --resource-group "az104-rg8" --show-details --output table
```

   <img src="capturas/t6-paso-03-az-vm-show.png" alt="az vm show" width="50%">

4. Desasigna:

```bash
az vm deallocate --resource-group "az104-rg8" --name "myCLIVM"
```

   <img src="capturas/t6-paso-04-vm-deallocated-cli.png" alt="VM deallocated CLI" width="50%">

**Resultado esperado:**
> `myCLIVM` en estado `Deallocated`. Tienes VMs creadas con cuatro métodos distintos en el mismo RG.

---

## ⚠️ Errores comunes

### Error 1 — `SkuNotAvailable` al desplegar el VMSS

**Síntoma:** Error indicando que el SKU Standard_D2s_v3 no está disponible en las zonas seleccionadas.

**Causa:** Algunos SKUs tienen disponibilidad limitada en ciertas zonas.

**Solución:**
```powershell
# Listar SKUs disponibles en la región con soporte de zonas
az vm list-skus --location "eastus" --size "Standard_D2s" --query "[?restrictions==[]]" --output table
```

---

### Error 2 — Las reglas de autoscaling no se activan

**Síntoma:** La CPU supera el 70% pero el VMSS no escala.

**Causa:** El cooldown (5 minutos) evita scaling inmediato. Además, las métricas tardan ~1 minuto en propagarse.

**Solución:** Espera al menos 10-15 minutos y verifica en **Monitoring → Metrics** que la CPU efectivamente supera el umbral sostenidamente.

---

## 🧹 Limpieza de recursos

> ⚠️ Las VMs y el VMSS generan costo mientras existen. Elimina el RG al terminar.

### Método B — CLI

```powershell
az group delete --name "az104-rg8" --yes --no-wait
```

### Método C — PowerShell

```powershell
Remove-AzResourceGroup -Name "az104-rg8" -Force -AsJob
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| 2× VM Standard_D2s_v3 | ~1 hora | ~$0.38 |
| VMSS (2 instancias base) | ~1 hora | ~$0.21 |
| Load Balancer Standard | ~1 hora | ~$0.025 |
| Discos Premium SSD | ~1 hora | ~$0.013 |
| **Total lab** | | **~$0.65** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| VM individual | Azure VM | EC2 | Compute Engine | Compute Instance |
| Grupos de escalado | VMSS | Auto Scaling Group | Managed Instance Group | Instance Pool |
| Zonas de disponibilidad | Availability Zones | Availability Zones | Zones | Availability Domains |
| Costo VM 2vCPU 8GB/hora | ~$0.096 | ~$0.096 (t3.large) | ~$0.095 | ~$0.064 |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- Las Availability Zones protegen contra fallos de datacenter — siempre despliega al menos 2 zonas para producción
- El escalado vertical requiere reinicio de la VM; el horizontal (VMSS) añade instancias sin downtime
- `Deallocated` ≠ `Stopped`: solo en deallocated paras el cargo de cómputo
- Los VMSS son la base de cualquier arquitectura cloud-native escalable en Azure

**Cómo se usa en labs futuros:**
> Las VMs creadas aquí son la referencia para el **Lab-10 Backup** donde se configuran políticas de respaldo y replicación para disaster recovery.

**Siguiente lab recomendado:**
👉 [Lab 09a — Web Apps PaaS](../../Lab-09a-PaaS-Web/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
