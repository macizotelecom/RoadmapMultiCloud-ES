# Virtual Networking — Lab 04

> Fase: F2 | Cert: AZ-104 | Lab: 04 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 04
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.00 USD (redes y DNS estáticas, coste mínimo)
**Región usada:** East US
**Duración estimada:** 50 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Crear VNet con subredes desde el portal](#tarea-1)
- [Tarea 2 — Crear VNet con plantilla ARM](#tarea-2)
- [Tarea 3 — Configurar ASG y NSG](#tarea-3)
- [Tarea 4 — Configurar zonas DNS públicas y privadas](#tarea-4)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Crear redes virtuales con subredes organizadas por función desde el portal y mediante plantillas ARM
- [ ] Implementar Application Security Groups (ASG) y Network Security Groups (NSG) para control de tráfico granular
- [ ] Configurar zonas DNS públicas con registros A y validar resolución de nombres
- [ ] Configurar zonas DNS privadas vinculadas a redes virtuales para resolución interna

**Habilidades del examen que cubre este lab:**
> **Configure and manage virtual networking (25–30%)**
> — Configure virtual networks and subnets
> — Configure network security groups and application security groups
> — Configure Azure DNS

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 03 — ARM/Bicep *(familiaridad con despliegue de plantillas)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- Azure CLI instalada (`az login`)

**Conocimientos previos:**
- Notación CIDR y subnetting básico
- Diferencia entre DNS público (resolución desde Internet) y DNS privado (resolución interna en VNet)
- Concepto de reglas stateful en firewalls de red

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| Virtual Network | Red principal del lab | $0.00 |
| Network Security Group | Control de tráfico por reglas | $0.00 |
| Application Security Group | Agrupación lógica de VMs para NSG | $0.00 |
| Azure DNS (zona pública) | Resolución de nombres desde Internet | ~$0.50/mes |
| Azure DNS (zona privada) | Resolución interna en VNet | ~$0.10/mes |

---

## 🏗️ Arquitectura del lab

```
Suscripción Azure
│
├── CoreServicesVnet (10.20.0.0/16) — East US
│   ├── SharedServicesSubnet (10.20.10.0/24)
│   │   └── NSG: myNSGSecure
│   │       ├── Regla inbound: AllowASG (puerto 80,443 desde asg-web)
│   │       └── Regla outbound: DenyInternetOutbound
│   └── DatabaseSubnet (10.20.20.0/24)
│
├── ManufacturingVnet (10.30.0.0/16) — East US
│   ├── SensorSubnet1 (10.30.20.0/24)
│   └── SensorSubnet2 (10.30.21.0/24)
│
├── ASG: asg-web
├── Zona DNS pública: contoso.com → registro A: www → 10.1.1.4
└── Zona DNS privada: private.contoso.com
    └── Vínculo: ManufacturingVnet
    └── Registro A: sensorvm → 10.1.1.4
```

**Lo que construimos en este lab:**
Dos redes virtuales con subredes segmentadas por función, un sistema de seguridad por capas con ASG+NSG, y resolución de nombres tanto pública como privada. Refleja una arquitectura empresarial real con separación de servicios centrales y manufactura.

---

## Tarea 1 — Crear VNet con subredes desde el portal

**Objetivo de la tarea:** Crear `CoreServicesVnet` con dos subredes funcionales y exportar su plantilla ARM para reutilizarla en la Tarea 2.

### Método A — Portal web

1. Portal → busca **Redes Virtuales** → **+ Crear**

   <img src="capturas/t1-paso-01-buscar-vnets.png" alt="Buscar VNets" width="50%">

2. Selecciona **Crear**

   <img src="capturas/t1-paso-02-crear-vnet.png" alt="Crear VNet" width="50%">

3. Configura los datos básicos de `CoreServicesVnet`:
   - **Resource Group:** `az104-rg4`
   - **Name:** `CoreServicesVnet`
   - **Region:** `East US`

   <img src="capturas/t1-paso-03-basics-coreservicesvnet.png" alt="Basics CoreServicesVnet" width="50%">

4. Pestaña **Address Space** → reemplaza con `10.20.0.0/16`

   <img src="capturas/t1-paso-04-address-space.png" alt="Address space" width="50%">

5. Agrega las subredes:
   - `SharedServicesSubnet` → `10.20.10.0/24`
   - `DatabaseSubnet` → `10.20.20.0/24`

   <img src="capturas/t1-paso-05-crear-subnet-shared.png" alt="Crear subnet Shared" width="50%">
   <img src="capturas/t1-paso-06-crear-subnet-database.png" alt="Crear subnet Database" width="50%">

6. **Revisar y crear**

   <img src="capturas/t1-paso-07-review-create.png" alt="Review create" width="50%">

7. Validación correcta → **Crear**

   <img src="capturas/t1-paso-08-validacion-ok.png" alt="Validación ok" width="50%">

8. Deployment completado

   <img src="capturas/t1-paso-09-deployment-ok.png" alt="Deployment ok" width="50%">

9. **Ir al recurso** → en **Automatización** → **Exportar plantilla** → **Descargar**

   <img src="capturas/t1-paso-10-ir-al-recurso.png" alt="Ir al recurso" width="50%">
   <img src="capturas/t1-paso-11-exportar-template.png" alt="Exportar template" width="50%">
   <img src="capturas/t1-paso-12-descargar-template-params.png" alt="Descargar template y params" width="50%">

### Método B — CLI

```powershell
az group create --name "az104-rg4" --location "eastus"

az network vnet create `
  --name "CoreServicesVnet" `
  --resource-group "az104-rg4" `
  --location "eastus" `
  --address-prefix "10.20.0.0/16"

az network vnet subnet create `
  --name "SharedServicesSubnet" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg4" `
  --address-prefix "10.20.10.0/24"

az network vnet subnet create `
  --name "DatabaseSubnet" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg4" `
  --address-prefix "10.20.20.0/24"
```

**Resultado esperado:**
> `CoreServicesVnet` con dos subredes creada. Tienes `template.json` y `parameters.json` descargados.

---

## Tarea 2 — Crear VNet con plantilla ARM

**Objetivo de la tarea:** Editar la plantilla exportada para crear `ManufacturingVnet` con diferente espacio de direcciones y subredes. Demuestra el patrón IaC de reutilización de plantillas.

### Método A — Portal web

1. Edita `template.json` — reemplaza `CoreServicesVnet` → `ManufacturingVnet` y `10.20.0.0` → `10.30.0.0`

   <img src="capturas/t2-paso-01-editar-manufacturing.png" alt="Editar manufacturing" width="50%">
   <img src="capturas/t2-paso-02-editar-address-space.png" alt="Editar address space" width="50%">

2. Renombra subredes: `SharedServicesSubnet` → `SensorSubnet1` con dirección `10.30.20.0/24`

   <img src="capturas/t2-paso-03-renombrar-subnet1.png" alt="Renombrar subnet1" width="50%">
   <img src="capturas/t2-paso-04-subnet1-address.png" alt="Subnet1 address" width="50%">

3. `DatabaseSubnet` → `SensorSubnet2` con dirección `10.30.21.0/24`

   <img src="capturas/t2-paso-05-renombrar-subnet2.png" alt="Renombrar subnet2" width="50%">
   <img src="capturas/t2-paso-06-subnet2-address.png" alt="Subnet2 address" width="50%">

4. Edita `parameters.json` → reemplaza `CoreServicesVnet` → `ManufacturingVnet`

   <img src="capturas/t2-paso-07-editar-parameters.png" alt="Editar parameters" width="50%">

5. Portal → **Implementar una plantilla personalizada** → **Cree su propia plantilla**

   <img src="capturas/t2-paso-08-custom-template.png" alt="Custom template" width="50%">

6. Carga y guarda el `template.json` editado

   <img src="capturas/t2-paso-09-cargar-template.png" alt="Cargar template" width="50%">
   <img src="capturas/t2-paso-10-template-guardado.png" alt="Template guardado" width="50%">

7. **Editar parámetros** → carga `parameters.json`

   <img src="capturas/t2-paso-11-editar-parameters.png" alt="Editar parameters portal" width="50%">
   <img src="capturas/t2-paso-12-parameters-cargados.png" alt="Parameters cargados" width="50%">

8. **Review + create** → **Create**

   <img src="capturas/t2-paso-13-review-create.png" alt="Review create" width="50%">
   <img src="capturas/t2-paso-14-deployment-iniciado.png" alt="Deployment iniciado" width="50%">

9. Verifica que `ManufacturingVnet` está creada con sus subredes

   <img src="capturas/t2-paso-15-manufacturing-creada.png" alt="Manufacturing creada" width="50%">
   <img src="capturas/t2-paso-16-subnets-manufacturing.png" alt="Subnets manufacturing" width="50%">

**Resultado esperado:**
> `ManufacturingVnet` (10.30.0.0/16) con `SensorSubnet1` y `SensorSubnet2` creadas correctamente.

---

## Tarea 3 — Configurar ASG y NSG

**Objetivo de la tarea:** Crear un Application Security Group para agrupar VMs web y un Network Security Group con reglas que permitan tráfico HTTP/HTTPS desde el ASG y denieguen salida a Internet. Los ASG eliminan la necesidad de gestionar IPs individuales en las reglas NSG.

### Método A — Portal web

1. Portal → busca **Application security groups** → **+ Crear**

   <img src="capturas/t3-paso-01-buscar-asg.png" alt="Buscar ASG" width="50%">
   <img src="capturas/t3-paso-02-crear-asg.png" alt="Crear ASG" width="50%">

2. Configura: **Name:** `asg-web` | **RG:** `az104-rg4` | **Region:** East US

   <img src="capturas/t3-paso-03-asg-basics.png" alt="ASG basics" width="50%">
   <img src="capturas/t3-paso-04-asg-creado.png" alt="ASG creado" width="50%">

3. Portal → **Network security groups** → **+ Crear**

   <img src="capturas/t3-paso-05-buscar-nsg.png" alt="Buscar NSG" width="50%">
   <img src="capturas/t3-paso-06-nsg-crear-recurso.png" alt="NSG crear recurso" width="50%">

4. Configura: **Name:** `myNSGSecure` | **RG:** `az104-rg4` | **Region:** East US

   <img src="capturas/t3-paso-07-nsg-basics.png" alt="NSG basics" width="50%">
   <img src="capturas/t3-paso-08-nsg-review-create.png" alt="NSG review create" width="50%">

5. Ir al recurso → **Configuración → Subredes** → **Asociar** → `CoreServicesVnet / SharedServicesSubnet`

   <img src="capturas/t3-paso-09-nsg-creado.png" alt="NSG creado" width="50%">
   <img src="capturas/t3-paso-10-nsg-asociar-subnet.png" alt="NSG asociar subnet" width="50%">
   <img src="capturas/t3-paso-11-nsg-subnet-asociada.png" alt="NSG subnet asociada" width="50%">

6. **Reglas de seguridad de entrada** → **+ Agregar** con:
   - Origen: Application security group → `asg-web`
   - Puertos destino: `80,443` | Protocolo: TCP | Acción: Allow | Prioridad: 100 | Nombre: `AllowASG`

   <img src="capturas/t3-paso-12-regla-inbound-asg.png" alt="Regla inbound ASG" width="50%">
   <img src="capturas/t3-paso-13-regla-inbound-guardada.png" alt="Regla inbound guardada" width="50%">

7. **Reglas de seguridad de salida** → **+ Agregar** con:
   - Destino: Service tag → Internet | Acción: Deny | Prioridad: 4096 | Nombre: `DenyInternetOutbound`

   <img src="capturas/t3-paso-14-regla-outbound-deny.png" alt="Regla outbound deny" width="50%">

### Método B — CLI

```powershell
# ASG
az network asg create `
  --name "asg-web" `
  --resource-group "az104-rg4" `
  --location "eastus"

# NSG
az network nsg create `
  --name "myNSGSecure" `
  --resource-group "az104-rg4" `
  --location "eastus"

# Asociar NSG a subred
az network vnet subnet update `
  --name "SharedServicesSubnet" `
  --vnet-name "CoreServicesVnet" `
  --resource-group "az104-rg4" `
  --network-security-group "myNSGSecure"

# Regla inbound desde ASG
az network nsg rule create `
  --name "AllowASG" `
  --nsg-name "myNSGSecure" `
  --resource-group "az104-rg4" `
  --priority 100 `
  --source-asgs "asg-web" `
  --destination-port-ranges 80 443 `
  --protocol TCP `
  --access Allow

# Regla outbound deny Internet
az network nsg rule create `
  --name "DenyInternetOutbound" `
  --nsg-name "myNSGSecure" `
  --resource-group "az104-rg4" `
  --priority 4096 `
  --destination-address-prefixes Internet `
  --direction Outbound `
  --access Deny
```

**Resultado esperado:**
> El NSG `myNSGSecure` está asociado a `SharedServicesSubnet` con las reglas `AllowASG` (inbound) y `DenyInternetOutbound` (outbound) configuradas.

---

## Tarea 4 — Configurar zonas DNS públicas y privadas

**Objetivo de la tarea:** Crear una zona DNS pública `contoso.com` con un registro A y validar la resolución. Luego crear una zona DNS privada `private.contoso.com` vinculada a `ManufacturingVnet` para resolución interna.

### Método A — Portal web

**DNS Pública:**

1. Portal → **Zona DNS** → **+ Crear**

   <img src="capturas/t4-paso-01-buscar-zona-dns.png" alt="Buscar zona DNS" width="50%">
   <img src="capturas/t4-paso-02-crear-zona-dns.png" alt="Crear zona DNS" width="50%">

2. **Name:** `contoso.com` | **RG:** `az104-rg4`

   <img src="capturas/t4-paso-03-dns-publico-basics.png" alt="DNS público basics" width="50%">
   <img src="capturas/t4-paso-04-dns-publico-review.png" alt="DNS público review" width="50%">

3. Zona creada — anota los 4 name servers

   <img src="capturas/t4-paso-05-dns-publico-creado.png" alt="DNS público creado" width="50%">
   <img src="capturas/t4-paso-06-nameservers.png" alt="Nameservers" width="50%">

4. **Administración de DNS → Registros** → **+ Agregar** un registro A:
   - **Name:** `www` | **Type:** A | **TTL:** 1 | **IP:** `10.1.1.4`

   <img src="capturas/t4-paso-07-registros-dns.png" alt="Registros DNS" width="50%">
   <img src="capturas/t4-paso-08-agregar-registro-a.png" alt="Agregar registro A" width="50%">

5. Valida la resolución (usa uno de los name servers de la zona):

```powershell
nslookup www.contoso.com <name-server-de-la-zona>
```

   <img src="capturas/t4-paso-09-nslookup-resultado.png" alt="nslookup resultado" width="50%">

**DNS Privada:**

6. Portal → **Zonas de DNS privado** → **+ Crear**

   <img src="capturas/t4-paso-10-buscar-zona-privada.png" alt="Buscar zona privada" width="50%">
   <img src="capturas/t4-paso-11-crear-zona-privada.png" alt="Crear zona privada" width="50%">

7. **Name:** `private.contoso.com` | **RG:** `az104-rg4`

   <img src="capturas/t4-paso-12-dns-privado-basics.png" alt="DNS privado basics" width="50%">
   <img src="capturas/t4-paso-13-dns-privado-review.png" alt="DNS privado review" width="50%">
   <img src="capturas/t4-paso-14-dns-privado-creado.png" alt="DNS privado creado" width="50%">

8. **Administración DNS → Vínculos de virtual network** → vincula `ManufacturingVnet`

   <img src="capturas/t4-paso-15-vnet-link.png" alt="VNet link" width="50%">

9. Agrega registro A: `sensorvm` → `10.1.1.4`

   <img src="capturas/t4-paso-16-registro-sensorvm.png" alt="Registro sensorvm" width="50%">

### Método B — CLI

```powershell
# Zona DNS pública
az network dns zone create `
  --name "contoso.com" `
  --resource-group "az104-rg4"

# Registro A
az network dns record-set a add-record `
  --zone-name "contoso.com" `
  --resource-group "az104-rg4" `
  --record-set-name "www" `
  --ipv4-address "10.1.1.4"

# Zona DNS privada
az network private-dns zone create `
  --name "private.contoso.com" `
  --resource-group "az104-rg4"

# Vincular a ManufacturingVnet
$vnetId = az network vnet show --name "ManufacturingVnet" --resource-group "az104-rg4" --query id --output tsv
az network private-dns link vnet create `
  --name "manufacturing-link" `
  --zone-name "private.contoso.com" `
  --resource-group "az104-rg4" `
  --virtual-network $vnetId `
  --registration-enabled false

# Registro A privado
az network private-dns record-set a add-record `
  --zone-name "private.contoso.com" `
  --resource-group "az104-rg4" `
  --record-set-name "sensorvm" `
  --ipv4-address "10.1.1.4"
```

**Resultado esperado:**
> La zona `contoso.com` tiene un registro A `www → 10.1.1.4` validado con nslookup. La zona `private.contoso.com` está vinculada a `ManufacturingVnet` con el registro `sensorvm`.

<img src="capturas/resultado-final-arquitectura.png" alt="Resultado final arquitectura" width="50%">

---

## ⚠️ Errores comunes

### Error 1 — Solapamiento de direcciones IP entre VNets

**Síntoma:** No puedes crear peering entre VNets más adelante (Lab-05).

**Causa:** Si dos VNets tienen rangos solapados (ej. ambas usan `10.0.0.0/16`) no pueden conectarse mediante peering.

**Solución:** Verifica que `CoreServicesVnet` use `10.20.0.0/16` y `ManufacturingVnet` use `10.30.0.0/16` sin solapamiento.

---

### Error 2 — La zona DNS privada no resuelve nombres

**Síntoma:** `nslookup sensorvm.private.contoso.com` no devuelve resultado desde una VM en ManufacturingVnet.

**Causa:** El vínculo de red virtual no tiene habilitado el **auto-registration** o la VM no está en la VNet vinculada.

**Solución:**
```powershell
# Verificar el estado del vínculo
az network private-dns link vnet show `
  --name "manufacturing-link" `
  --zone-name "private.contoso.com" `
  --resource-group "az104-rg4"
```

---

## 🧹 Limpieza de recursos

### Método A — Portal

Navega a `az104-rg4` → **Eliminar grupo de recursos**

<img src="capturas/limpieza-eliminar-rg.png" alt="Limpieza" width="50%">

### Método B — CLI

```powershell
az group delete --name "az104-rg4" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| Virtual Networks (2) | ~1 hora | $0.00 |
| NSG, ASG | ~1 hora | $0.00 |
| Zona DNS pública | ~1 hora | ~$0.001 |
| Zona DNS privada | ~1 hora | ~$0.0001 |
| **Total lab** | | **~$0.00** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Red virtual | VNet | VPC | VPC | VCN |
| Control de tráfico | NSG + ASG | Security Groups + NACLs | Firewall Rules | Security Lists + NSG |
| DNS público | Azure DNS | Route 53 | Cloud DNS | OCI DNS |
| DNS privado | Private DNS Zone | Route 53 Private Hosted Zone | Cloud DNS Private Zone | Private DNS |
| Costo VNet | $0.00 | $0.00 | $0.00 | $0.00 |
| Costo DNS público/mes | $0.50/zona | $0.50/zona | $0.20/zona | $0.70/zona |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- El diseño de red con subredes funcionales (no por tecnología sino por rol: servicios compartidos, manufactura, sensores) es la práctica estándar en Azure
- ASG + NSG es la combinación correcta en producción: el ASG agrupa VMs por función, el NSG define las reglas — si una VM cambia de rol, solo cambias su ASG membership
- DNS público y privado en Azure son servicios separados con propósitos distintos — el privado no es visible desde Internet

**Cómo se usa en labs futuros:**
> Las VNets `CoreServicesVnet` y `ManufacturingVnet` se conectan en el **Lab-05 Intersite** mediante VNet Peering para habilitar comunicación entre ellas.

**Siguiente lab recomendado:**
👉 [Lab 05 — Intersite Connectivity](../../Lab-05-Intersite/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
