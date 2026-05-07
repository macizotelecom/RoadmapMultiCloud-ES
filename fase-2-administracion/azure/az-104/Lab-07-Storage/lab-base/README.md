# Azure Storage — Lab 07

> Fase: F2 | Cert: AZ-104 | Lab: 07 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 07
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.10 USD/hora (Storage cuenta GRS + File share)
**Región usada:** East US
**Duración estimada:** 45 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Crear y configurar cuenta de almacenamiento](#tarea-1)
- [Tarea 2 — Blob Storage seguro con SAS y retención inmutable](#tarea-2)
- [Tarea 3 — Azure File Share con restricción de red](#tarea-3)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Crear una cuenta de almacenamiento con redundancia GRS y configurar lifecycle management
- [ ] Crear contenedores Blob con políticas de retención inmutable y acceso via SAS tokens
- [ ] Crear y administrar Azure File Shares usando Storage Browser
- [ ] Restringir el acceso a la cuenta de almacenamiento mediante service endpoints y VNet

**Habilidades del examen que cubre este lab:**
> **Implement and manage storage (15–20%)**
> — Configure Azure Storage accounts
> — Configure Azure Blob Storage
> — Configure Azure Files
> — Configure storage security

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 04 — Virtual Networking *(para la restricción de red de la Tarea 3)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor**
- Azure CLI instalada

**Conocimientos previos:**
- Diferencia entre Blob (datos no estructurados), Files (compartidos SMB) y Queue/Table (datos estructurados)
- SAS token: URL firmada con tiempo de expiración que permite acceso temporal sin compartir claves
- GRS replica datos en dos regiones — LRS solo en una

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| Storage Account (GRS Standard) | Cuenta principal del lab | ~$0.046/GB/mes |
| Blob Container | Almacenamiento de objetos con retención | ~$0.018/GB/mes (Hot) |
| Azure File Share | Compartido de archivos SMB | ~$0.06/GB/mes |
| Virtual Network + Service Endpoint | Restricción de acceso por red | $0.00 |

---

## 🏗️ Arquitectura del lab

```
Storage Account: stlab07<sufijo> (GRS, East US)
│
├── Blob Container: data (acceso privado)
│   ├── Política retención inmutable: 180 días
│   ├── Carpeta: securitytest/
│   │   └── archivo-subido.txt
│   └── Acceso: solo via SAS token
│
├── File Share: share1 (Transaction Optimized)
│   └── Administrado via Storage Browser
│
└── Networking: solo desde vnet1/default subnet
    └── Service Endpoint: Microsoft.Storage
```

---

## Tarea 1 — Crear y configurar cuenta de almacenamiento

**Objetivo de la tarea:** Crear una cuenta de almacenamiento con redundancia geográfica, acceso público deshabilitado inicialmente y reglas de lifecycle para mover datos al tier Cool automáticamente.

### Método A — Portal web

1. Portal → **Storage accounts** → **+ Create**

   ![Buscar Storage accounts](capturas/t1-paso-01-buscar-storage-accounts.png)
   ![Crear storage account](capturas/t1-paso-02-crear-storage-account.png)

2. **Basics**:
   - **RG:** `az104-rg7` | **Name:** nombre único global (3-24 chars, minúsculas)
   - **Region:** East US | **Performance:** Standard | **Redundancy:** GRS
   - Activa: *Make read access available in the event of regional unavailability*

   ![Basics GRS config](capturas/t1-paso-03-basics-grs-config.png)

3. **Review + create** → **Create** → **Go to resource**

   ![Review create](capturas/t1-paso-04-review-create.png)
   ![Ir al recurso](capturas/t1-paso-05-ir-al-recurso.png)

4. **Security + networking → Networking** → verifica que el acceso público está deshabilitado

   ![Networking disable](capturas/t1-paso-06-networking-disable.png)

5. **Manage** → cambia a **Enabled from selected networks** → añade tu IP actual

   ![Habilitar redes seleccionadas](capturas/t1-paso-07-habilitar-redes-seleccionadas.png)
   ![Agregar IP cliente](capturas/t1-paso-08-agregar-ip-cliente.png)

6. **Data management → Redundancy** → revisa los centros de datos primario y secundario

   ![Redundancia GRS](capturas/t1-paso-09-redundancia-grs.png)

7. **Lifecycle management** → **+ Add a rule** → nombre: `Movetocool`:
   - Condición: blob no modificado en 30 días → mover a Cool tier

   ![Lifecycle management](capturas/t1-paso-10-lifecycle-management.png)
   ![Nueva regla lifecycle](capturas/t1-paso-11-nueva-regla-lifecycle.png)
   ![Regla cool tier](capturas/t1-paso-12-regla-cool-tier.png)
   ![Regla guardada](capturas/t1-paso-13-regla-guardada.png)

### Método B — CLI

```powershell
az group create --name "az104-rg7" --location "eastus"

$stName = "stlab07$(Get-Random -Maximum 9999)"
az storage account create `
  --name $stName `
  --resource-group "az104-rg7" `
  --location "eastus" `
  --sku "Standard_GRS" `
  --allow-blob-public-access false

# Regla de lifecycle
$policy = '{"rules":[{"name":"Movetocool","type":"Lifecycle","definition":{"actions":{"baseBlob":{"tierToCool":{"daysAfterModificationGreaterThan":30}}},"filters":{"blobTypes":["blockBlob"]}}}]}'
az storage account management-policy create --account-name $stName --resource-group "az104-rg7" --policy $policy
```

**Resultado esperado:**
> Cuenta de almacenamiento GRS con acceso desde tu IP, redundancia geográfica y regla lifecycle configurada.

---

## Tarea 2 — Blob Storage seguro con SAS y retención inmutable

**Objetivo de la tarea:** Crear un contenedor con política de retención inmutable (WORM: Write Once Read Many), subir un archivo y verificar que sin SAS el acceso es denegado, con SAS es permitido.

### Método A — Portal web

1. **Data storage → Containers** → **+ Add container**:
   - **Name:** `data` | **Access:** Private

   ![Containers menu](capturas/t2-paso-01-containers-menu.png)
   ![Crear container data](capturas/t2-paso-02-crear-container-data.png)

2. En el contenedor `data` → menú `...` → **Access policy** → **Immutable blob storage → + Add policy**:
   - **Policy type:** Time-based retention | **Retain for:** 180 días

   ![Access policy menu](capturas/t2-paso-03-access-policy-menu.png)
   ![Immutable retention policy](capturas/t2-paso-04-immutable-retention-policy.png)

3. **Upload** → expande **Advanced**:
   - Blob type: Block blob | Access tier: Hot | Upload to folder: `securitytest`

   ![Subir archivo advanced](capturas/t2-paso-05-subir-archivo-advanced.png)
   ![Archivo subido](capturas/t2-paso-06-archivo-subido.png)

4. Copia la URL del archivo → ábrela en ventana incógnito → verifica error `ResourceNotFound`

   ![Acceso denegado sin SAS](capturas/t2-paso-07-acceso-denegado-sin-sas.png)

5. Menú `...` del archivo → **Generate SAS**:
   - **Permissions:** Read | **Expiry:** mañana | **Signing key:** Key 1

   ![Generar SAS](capturas/t2-paso-08-generar-sas.png)
   ![Configurar SAS](capturas/t2-paso-09-configurar-sas.png)

6. Copia el **Blob SAS URL** → ábrelo en ventana incógnito → el archivo debe ser visible

   ![Archivo visible con SAS](capturas/t2-paso-10-archivo-visible-con-sas.png)

### Método B — CLI

```powershell
$stName = az storage account list --resource-group "az104-rg7" --query "[0].name" --output tsv
$key = az storage account keys list --account-name $stName --resource-group "az104-rg7" --query "[0].value" --output tsv

# Crear contenedor
az storage container create --name "data" --account-name $stName --account-key $key --public-access off

# Subir archivo
az storage blob upload --account-name $stName --account-key $key --container-name "data" --name "securitytest/test.txt" --data "archivo de prueba" --overwrite

# Generar SAS token (válido 1 hora)
$expiry = (Get-Date).AddHours(1).ToString("yyyy-MM-ddTHH:mmZ")
$sas = az storage blob generate-sas --account-name $stName --account-key $key --container-name "data" --name "securitytest/test.txt" --permissions r --expiry $expiry --output tsv
$url = az storage blob url --account-name $stName --account-key $key --container-name "data" --name "securitytest/test.txt" --output tsv
Write-Host "URL con SAS: $url?$sas"
```

**Resultado esperado:**
> Sin SAS: acceso denegado. Con SAS: archivo accesible durante el período configurado.

---

## Tarea 3 — Azure File Share con restricción de red

**Objetivo de la tarea:** Crear un File Share administrado desde Storage Browser y luego restringir el acceso a la cuenta de almacenamiento para que solo sea accesible desde una VNet específica.

### Método A — Portal web

1. **Data storage → File shares** → **+ File share**:
   - **Name:** `share1` | **Access tier:** Transaction optimized

   ![File shares menu](capturas/t3-paso-01-file-shares-menu.png)
   ![Crear file share](capturas/t3-paso-02-crear-file-share.png)
   ![Share1 basics](capturas/t3-paso-03-share1-basics.png)
   ![Share1 desplegado](capturas/t3-paso-04-share1-desplegado.png)

2. **Storage browser** → **File shares** → verifica `share1`

   ![Storage browser](capturas/t3-paso-05-storage-browser.png)
   ![Share1 en browser](capturas/t3-paso-06-share1-en-browser.png)

3. Portal → **Virtual networks** → **+ Create**: nombre `vnet1` | RG `az104-rg7`

   ![Buscar VNets](capturas/t3-paso-07-buscar-vnets.png)
   ![Crear vnet1](capturas/t3-paso-08-crear-vnet1.png)
   ![VNet1 review create](capturas/t3-paso-09-vnet1-review-create.png)

4. En `vnet1` → **Settings → Service endpoints** → **+ Add**:
   - **Service:** `Microsoft.Storage` | **Subnets:** default

   ![Service endpoints](capturas/t3-paso-10-service-endpoints.png)
   ![Agregar storage endpoint](capturas/t3-paso-11-agregar-storage-endpoint.png)

5. Vuelve a la Storage Account → **Networking → Manage** → **+ Add a virtual network**:
   - Selecciona `vnet1` y la subred `default`

   ![Networking manage](capturas/t3-paso-12-networking-manage.png)
   ![Agregar vnet1](capturas/t3-paso-13-agregar-vnet1.png)

6. Elimina tu IP del listado → Guarda → vuelve a Storage Browser → recarga → verifica acceso denegado

   ![Acceso no autorizado](capturas/t3-paso-14-acceso-no-autorizado.png)

### Método B — CLI

```powershell
# Crear VNet con service endpoint
az network vnet create --name "vnet1" --resource-group "az104-rg7" --location "eastus"
az network vnet subnet update --name "default" --vnet-name "vnet1" --resource-group "az104-rg7" --service-endpoints "Microsoft.Storage"

# Obtener ID de subred
$subnetId = az network vnet subnet show --name "default" --vnet-name "vnet1" --resource-group "az104-rg7" --query id --output tsv

# Restringir storage account a la VNet
az storage account network-rule add --account-name $stName --resource-group "az104-rg7" --subnet $subnetId
az storage account update --name $stName --resource-group "az104-rg7" --default-action Deny
```

**Resultado esperado:**
> Storage Browser muestra error de acceso no autorizado al intentar navegar el File Share, confirmando que la restricción de red funciona.

---

## ⚠️ Errores comunes

### Error 1 — No puedo subir archivos al contenedor desde el portal

**Síntoma:** El portal muestra error al intentar subir después de configurar la restricción de red.

**Causa:** Al añadir la VNet y quitar tu IP, bloqueaste también tu propio acceso desde el portal.

**Solución:** Añade tu IP actual de nuevo antes de trabajar con el storage:
```powershell
$myIp = (Invoke-WebRequest -Uri "https://api.ipify.org" -UseBasicParsing).Content
az storage account network-rule add --account-name $stName --resource-group "az104-rg7" --ip-address $myIp
```

---

### Error 2 — El SAS token devuelve 403 Forbidden

**Síntoma:** La URL con SAS devuelve error 403 en lugar de mostrar el archivo.

**Causa:** El SAS token expiró, o la **Start date** es una fecha futura (por zona horaria).

**Solución:** Al generar el SAS, pon la **Start date** en el día de ayer para evitar problemas de reloj.

---

## 🧹 Limpieza de recursos

### Método A — Portal

`az104-rg7` → **Eliminar grupo de recursos**

### Método B — CLI

```powershell
az group delete --name "az104-rg7" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| Storage Account GRS (datos mínimos) | ~1 hora | ~$0.001 |
| File Share (Transaction Optimized) | ~1 hora | ~$0.001 |
| VNet + Service Endpoint | ~1 hora | $0.00 |
| **Total lab** | | **~$0.01** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Almacenamiento objetos | Blob Storage | S3 | Cloud Storage | Object Storage |
| Compartido de archivos SMB | Azure Files | FSx for Windows | Filestore | File Storage |
| Redundancia geográfica | GRS / GZRS | S3 Cross-Region Replication | Multi-region | Cross-Region Replication |
| Acceso temporal seguro | SAS Token | Presigned URL | Signed URL | Pre-Authenticated Request |
| Costo blob Hot/GB/mes | $0.018 | $0.023 | $0.020 | $0.0255 |
| Costo blob Cool/GB/mes | $0.010 | $0.0125 | $0.010 | $0.013 |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- GRS vs LRS: GRS protege contra desastre regional completo — úsalo para datos críticos que no puedes perder
- SAS tokens son el mecanismo correcto para dar acceso temporal a objetos sin exponer las claves de cuenta
- Las políticas de retención inmutable (WORM) son obligatorias en sectores regulados (financiero, salud)
- Service endpoints permiten que el tráfico de Azure hacia Storage no salga a Internet público

**Cómo se usa en labs futuros:**
> El Storage Account creado aquí sirve como destino de logs de diagnóstico en el **Lab-10 Backup** y **Lab-11 Monitoring**.

**Siguiente lab recomendado:**
👉 [Lab 08 — Virtual Machines](../../Lab-08-VMs/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
