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

   ![Buscar VNets](capturas/t1-paso-01-buscar-vnets.png)

2. Selecciona **Crear**

   ![Crear VNet](capturas/t1-paso-02-crear-vnet.png)

3. Configura los datos básicos de `CoreServicesVnet`:
   - **Resource Group:** `az104-rg4`
   - **Name:** `CoreServicesVnet`
   - **Region:** `East US`

   ![Basics CoreServicesVnet](capturas/t1-paso-03-basics-coreservicesvnet.png)

4. Pestaña **Address Space** → reemplaza con `10.20.0.0/16`

   ![Address space](capturas/t1-paso-04-address-space.png)

5. Agrega las subredes:
   - `SharedServicesSubnet` → `10.20.10.0/24`
   - `DatabaseSubnet` → `10.20.20.0/24`

   ![Crear subnet Shared](capturas/t1-paso-05-crear-subnet-shared.png)
   ![Crear subnet Database](capturas/t1-paso-06-crear-subnet-database.png)

6. **Revisar y crear**

   ![Review create](capturas/t1-paso-07-review-create.png)

7. Validación correcta → **Crear**

   ![Validación ok](capturas/t1-paso-08-validacion-ok.png)

8. Deployment completado

   ![Deployment ok](capturas/t1-paso-09-deployment-ok.png)

9. **Ir al recurso** → en **Automatización** → **Exportar plantilla** → **Descargar**

   ![Ir al recurso](capturas/t1-paso-10-ir-al-recurso.png)
   ![Exportar template](capturas/t1-paso-11-exportar-template.png)
   ![Descargar template y params](capturas/t1-paso-12-descargar-template-params.png)

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

   ![Editar manufacturing](capturas/t2-paso-01-editar-manufacturing.png)
   ![Editar address space](capturas/t2-paso-02-editar-address-space.png)

2. Renombra subredes: `SharedServicesSubnet` → `SensorSubnet1` con dirección `10.30.20.0/24`

   ![Renombrar subnet1](capturas/t2-paso-03-renombrar-subnet1.png)
   ![Subnet1 address](capturas/t2-paso-04-subnet1-address.png)

3. `DatabaseSubnet` → `SensorSubnet2` con dirección `10.30.21.0/24`

   ![Renombrar subnet2](capturas/t2-paso-05-renombrar-subnet2.png)
   ![Subnet2 address](capturas/t2-paso-06-subnet2-address.png)

4. Edita `parameters.json` → reemplaza `CoreServicesVnet` → `ManufacturingVnet`

   ![Editar parameters](capturas/t2-paso-07-editar-parameters.png)

5. Portal → **Implementar una plantilla personalizada** → **Cree su propia plantilla**

   ![Custom template](capturas/t2-paso-08-custom-template.png)

6. Carga y guarda el `template.json` editado

   ![Cargar template](capturas/t2-paso-09-cargar-template.png)
   ![Template guardado](capturas/t2-paso-10-template-guardado.png)

7. **Editar parámetros** → carga `parameters.json`

   ![Editar parameters portal](capturas/t2-paso-11-editar-parameters.png)
   ![Parameters cargados](capturas/t2-paso-12-parameters-cargados.png)

8. **Review + create** → **Create**

   ![Review create](capturas/t2-paso-13-review-create.png)
   ![Deployment iniciado](capturas/t2-paso-14-deployment-iniciado.png)

9. Verifica que `ManufacturingVnet` está creada con sus subredes

   ![Manufacturing creada](capturas/t2-paso-15-manufacturing-creada.png)
   ![Subnets manufacturing](capturas/t2-paso-16-subnets-manufacturing.png)

**Resultado esperado:**
> `ManufacturingVnet` (10.30.0.0/16) con `SensorSubnet1` y `SensorSubnet2` creadas correctamente.

---

## Tarea 3 — Configurar ASG y NSG

**Objetivo de la tarea:** Crear un Application Security Group para agrupar VMs web y un Network Security Group con reglas que permitan tráfico HTTP/HTTPS desde el ASG y denieguen salida a Internet. Los ASG eliminan la necesidad de gestionar IPs individuales en las reglas NSG.

### Método A — Portal web

1. Portal → busca **Application security groups** → **+ Crear**

   ![Buscar ASG](capturas/t3-paso-01-buscar-asg.png)
   ![Crear ASG](capturas/t3-paso-02-crear-asg.png)

2. Configura: **Name:** `asg-web` | **RG:** `az104-rg4` | **Region:** East US

   ![ASG basics](capturas/t3-paso-03-asg-basics.png)
   ![ASG creado](capturas/t3-paso-04-asg-creado.png)

3. Portal → **Network security groups** → **+ Crear**

   ![Buscar NSG](capturas/t3-paso-05-buscar-nsg.png)
   ![NSG crear recurso](capturas/t3-paso-06-nsg-crear-recurso.png)

4. Configura: **Name:** `myNSGSecure` | **RG:** `az104-rg4` | **Region:** East US

   ![NSG basics](capturas/t3-paso-07-nsg-basics.png)
   ![NSG review create](capturas/t3-paso-08-nsg-review-create.png)

5. Ir al recurso → **Configuración → Subredes** → **Asociar** → `CoreServicesVnet / SharedServicesSubnet`

   ![NSG creado](capturas/t3-paso-09-nsg-creado.png)
   ![NSG asociar subnet](capturas/t3-paso-10-nsg-asociar-subnet.png)
   ![NSG subnet asociada](capturas/t3-paso-11-nsg-subnet-asociada.png)

6. **Reglas de seguridad de entrada** → **+ Agregar** con:
   - Origen: Application security group → `asg-web`
   - Puertos destino: `80,443` | Protocolo: TCP | Acción: Allow | Prioridad: 100 | Nombre: `AllowASG`

   ![Regla inbound ASG](capturas/t3-paso-12-regla-inbound-asg.png)
   ![Regla inbound guardada](capturas/t3-paso-13-regla-inbound-guardada.png)

7. **Reglas de seguridad de salida** → **+ Agregar** con:
   - Destino: Service tag → Internet | Acción: Deny | Prioridad: 4096 | Nombre: `DenyInternetOutbound`

   ![Regla outbound deny](capturas/t3-paso-14-regla-outbound-deny.png)

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

   ![Buscar zona DNS](capturas/t4-paso-01-buscar-zona-dns.png)
   ![Crear zona DNS](capturas/t4-paso-02-crear-zona-dns.png)

2. **Name:** `contoso.com` | **RG:** `az104-rg4`

   ![DNS público basics](capturas/t4-paso-03-dns-publico-basics.png)
   ![DNS público review](capturas/t4-paso-04-dns-publico-review.png)

3. Zona creada — anota los 4 name servers

   ![DNS público creado](capturas/t4-paso-05-dns-publico-creado.png)
   ![Nameservers](capturas/t4-paso-06-nameservers.png)

4. **Administración de DNS → Registros** → **+ Agregar** un registro A:
   - **Name:** `www` | **Type:** A | **TTL:** 1 | **IP:** `10.1.1.4`

   ![Registros DNS](capturas/t4-paso-07-registros-dns.png)
   ![Agregar registro A](capturas/t4-paso-08-agregar-registro-a.png)

5. Valida la resolución (usa uno de los name servers de la zona):

```powershell
nslookup www.contoso.com <name-server-de-la-zona>
```

   ![nslookup resultado](capturas/t4-paso-09-nslookup-resultado.png)

**DNS Privada:**

6. Portal → **Zonas de DNS privado** → **+ Crear**

   ![Buscar zona privada](capturas/t4-paso-10-buscar-zona-privada.png)
   ![Crear zona privada](capturas/t4-paso-11-crear-zona-privada.png)

7. **Name:** `private.contoso.com` | **RG:** `az104-rg4`

   ![DNS privado basics](capturas/t4-paso-12-dns-privado-basics.png)
   ![DNS privado review](capturas/t4-paso-13-dns-privado-review.png)
   ![DNS privado creado](capturas/t4-paso-14-dns-privado-creado.png)

8. **Administración DNS → Vínculos de virtual network** → vincula `ManufacturingVnet`

   ![VNet link](capturas/t4-paso-15-vnet-link.png)

9. Agrega registro A: `sensorvm` → `10.1.1.4`

   ![Registro sensorvm](capturas/t4-paso-16-registro-sensorvm.png)

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

![Resultado final arquitectura](capturas/resultado-final-arquitectura.png)

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

![Limpieza](capturas/limpieza-eliminar-rg.png)

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
