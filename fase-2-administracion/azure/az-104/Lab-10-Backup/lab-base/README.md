# Data Protection y Backup — Lab 10

> Fase: F2 | Cert: AZ-104 | Lab: 10 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 10
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.40 USD/hora (VM + Backup + Site Recovery)
**Región usada:** East US → West US (replicación)
**Duración estimada:** 60 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Provisionar infraestructura con plantilla](#tarea-1)
- [Tarea 2 — Crear y configurar Recovery Services Vault](#tarea-2)
- [Tarea 3 — Configurar backup de VM](#tarea-3)
- [Tarea 4 — Monitorear Azure Backup](#tarea-4)
- [Tarea 5 — Habilitar replicación con Site Recovery](#tarea-5)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Crear un Recovery Services Vault con redundancia GRS y soft delete configurado
- [ ] Definir políticas de backup (frecuencia, retención) y aplicarlas a una VM
- [ ] Configurar diagnósticos del Vault hacia una Storage Account para auditoría
- [ ] Habilitar replicación de VM hacia una región secundaria con Azure Site Recovery

**Habilidades del examen que cubre este lab:**
> **Monitor and back up Azure resources (10–15%)**
> — Configure Azure Backup
> — Configure Azure Site Recovery

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 03 — ARM/Bicep *(despliegue de plantillas)*
- [ ] Lab 07 — Storage *(Storage Account para diagnósticos)*

**Conocimientos previos:**
- Diferencia entre Backup (protección contra pérdida de datos) y Site Recovery (continuidad ante desastre regional)
- RPO (Recovery Point Objective) vs RTO (Recovery Time Objective)
- Soft delete: los datos eliminados se retienen 14 días extra para recuperación accidental

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| VM Standard_D2s_v3 | VM a proteger | ~$0.096/hora |
| Recovery Services Vault (East US) | Backup y Site Recovery | ~$10/VM/mes |
| Recovery Services Vault (West US) | Destino de replicación | ~$10/VM/mes |
| Storage Account | Destino de logs de diagnóstico | ~$0.002/hora |
| Azure Site Recovery | Replicación entre regiones | ~$25/VM protegida/mes |

---

## 🏗️ Arquitectura del lab

```
East US
├── Resource Group: az104-rg-region1
│   ├── VM: az104-10-vm0 (Windows Server)
│   ├── RSV: az104-rsv-region1 (GRS, Soft Delete 14d)
│   │   ├── Backup Policy: az104-backup (diario, 12AM)
│   │   └── Protected Item: az104-10-vm0
│   └── Storage Account: stdiag<sufijo>
│       └── Diagnostic Settings → Backup Logs
│
West US
└── Resource Group: az104-rg-region2
    └── RSV: az104-rsv-region2
        └── Replicated Item: az104-10-vm0 (via ASR)
```

---

## Tarea 1 — Provisionar infraestructura con plantilla

**Objetivo de la tarea:** Desplegar la VM que se va a proteger usando la plantilla oficial del lab.

### Método A — Portal web

1. Descarga `az104-10-vms-edge-template.json` desde la web oficial del lab

   <img src="capturas/t1-paso-01-descarga-lab10.png" alt="Descarga lab10" width="50%">

2. Portal → **Deploy a custom template** → **Build your own template**

   <img src="capturas/t1-paso-02-custom-template.png" alt="Custom template" width="50%">
   <img src="capturas/t1-paso-03-build-template.png" alt="Build template" width="50%">

3. **Load file** → selecciona el template

   <img src="capturas/t1-paso-04-load-file.png" alt="Load file" width="50%">
   <img src="capturas/t1-paso-05-seleccionar-template.png" alt="Seleccionar template" width="50%">

4. **Edit parameters** → carga el archivo de parámetros → configura password

   <img src="capturas/t1-paso-06-edit-parameters.png" alt="Edit parameters" width="50%">
   <img src="capturas/t1-paso-07-parameters-cargados.png" alt="Parameters cargados" width="50%">

5. **RG:** `az104-rg-region1` | **Region:** East US | **Review + create** → **Create**

   <img src="capturas/t1-paso-08-deployment-config.png" alt="Deployment config" width="50%">
   <img src="capturas/t1-paso-09-deployment-completado.png" alt="Deployment completado" width="50%">

**Resultado esperado:**
> VM `az104-10-vm0` desplegada en `az104-rg-region1` con su VNet.

---

## Tarea 2 — Crear y configurar Recovery Services Vault

**Objetivo de la tarea:** El RSV es el contenedor que almacena backups y gestiona la replicación. La configuración GRS + soft delete + Cross Region Restore es el estándar para entornos de producción.

### Método A — Portal web

1. Portal → **Recovery Services vaults** → **+ Create**

   <img src="capturas/t2-paso-01-buscar-rsv.png" alt="Buscar RSV" width="50%">
   <img src="capturas/t2-paso-02-crear-rsv.png" alt="Crear RSV" width="50%">

2. Configura:
   - **RG:** `az104-rg-region1` | **Name:** `az104-rsv-region1` | **Region:** East US

   <img src="capturas/t2-paso-03-rsv-basics.png" alt="RSV basics" width="50%">

3. **Go to resource** → **Settings → Properties**

   <img src="capturas/t2-paso-04-rsv-properties.png" alt="RSV properties" width="50%">

4. **Backup Configuration → Update** → verifica **Geo-redundant** y cierra

   <img src="capturas/t2-paso-05-backup-configuration.png" alt="Backup configuration" width="50%">
   <img src="capturas/t2-paso-06-geo-redundant.png" alt="Geo-redundant" width="50%">

5. **Security Settings → Soft Delete → Update** → verifica retención de **14 días**

   <img src="capturas/t2-paso-07-soft-delete-settings.png" alt="Soft delete settings" width="50%">
   <img src="capturas/t2-paso-08-14-dias-retencion.png" alt="14 días retención" width="50%">

### Método B — CLI

```powershell
az backup vault create `
  --name "az104-rsv-region1" `
  --resource-group "az104-rg-region1" `
  --location "eastus"

# Configurar GRS
az backup vault backup-properties set `
  --name "az104-rsv-region1" `
  --resource-group "az104-rg-region1" `
  --backup-storage-redundancy GeoRedundant
```

**Resultado esperado:**
> RSV `az104-rsv-region1` con redundancia GRS y soft delete de 14 días configurados.

---

## Tarea 3 — Configurar backup de VM

**Objetivo de la tarea:** Crear una política de backup y aplicarla a la VM. La política define cuándo se hacen los backups y cuánto tiempo se retienen.

### Método A — Portal web

1. En el RSV → **Overview → + Backup**

   <img src="capturas/t3-paso-01-backup-overview.png" alt="Backup overview" width="50%">

2. **Where is your workload:** Azure | **What to backup:** Virtual machine → **Backup**

   <img src="capturas/t3-paso-02-backup-goal.png" alt="Backup goal" width="50%">

3. **Policy subtype:** Standard → **Create a new policy**:
   - **Name:** `az104-backup` | **Frequency:** Daily | **Time:** 12:00 AM
   - **Retain instant recovery snapshot:** 2 días

   <img src="capturas/t3-paso-03-policy-type.png" alt="Policy type" width="50%">
   <img src="capturas/t3-paso-04-nueva-politica.png" alt="Nueva política" width="50%">

4. **Virtual Machines → + Add** → selecciona `az104-10-vm0` → **OK**

   <img src="capturas/t3-paso-05-agregar-vm.png" alt="Agregar VM" width="50%">
   <img src="capturas/t3-paso-06-seleccionar-vm.png" alt="Seleccionar VM" width="50%">

5. **Enable backup** → espera ~2 minutos

   <img src="capturas/t3-paso-07-habilitar-backup.png" alt="Habilitar backup" width="50%">
   <img src="capturas/t3-paso-08-backup-items.png" alt="Backup habilitado" width="50%">

6. **Protected items → Backup items → Azure Virtual Machine**

   <img src="capturas/t3-paso-09-azure-vm-backup.png" alt="Azure VM backup" width="50%">
   <img src="capturas/t3-paso-10-vm0-detalle.png" alt="VM0 detalle" width="50%">

7. **Backup now** → acepta fecha predeterminada → **OK**

   <img src="capturas/t3-paso-11-backup-now.png" alt="Backup now" width="50%">
   <img src="capturas/t3-paso-12-confirmar-backup.png" alt="Confirmar backup" width="50%">

**Resultado esperado:**
> La VM `az104-10-vm0` aparece en **Backup Items** con el primer backup en estado Initial Backup.

---

## Tarea 4 — Monitorear Azure Backup

**Objetivo de la tarea:** Enviar los logs y métricas del RSV a una Storage Account para auditoría y análisis posterior. En producción se suelen enviar a Log Analytics para consultas KQL.

### Método A — Portal web

1. Portal → **Storage accounts** → **+ Create** en `az104-rg-region1`

   <img src="capturas/t4-paso-01-buscar-storage-accounts.png" alt="Buscar Storage accounts" width="50%">
   <img src="capturas/t4-paso-02-crear-storage.png" alt="Crear storage" width="50%">
   <img src="capturas/t4-paso-03-storage-config.png" alt="Storage config" width="50%">

2. Vuelve al RSV → **Monitoring → Diagnostic Settings** → **+ Add diagnostic setting**

   <img src="capturas/t4-paso-04-diagnostic-settings.png" alt="Diagnostic settings" width="50%">
   <img src="capturas/t4-paso-05-add-diagnostic.png" alt="Add diagnostic" width="50%">

3. Nombre: `Logs and Metrics to storage`

   <img src="capturas/t4-paso-06-nombre-diagnostic.png" alt="Nombre diagnostic" width="50%">

4. Marca las categorías:
   - Azure Backup Reporting Data, Addon Azure Backup Job Data, Addon Azure Backup Alert Data
   - Azure Site Recovery Jobs, Azure Site Recovery Events
   - **Destination:** Archive to a storage account → selecciona la Storage Account creada

   <img src="capturas/t4-paso-07-categorias-logs.png" alt="Categorías logs" width="50%">

5. **Save** → **Monitoring → Backup jobs** → verifica el job de `az104-10-vm0`

   <img src="capturas/t4-paso-08-backup-jobs-menu.png" alt="Backup jobs menú" width="50%">
   <img src="capturas/t4-paso-09-job-vm0.png" alt="Job vm0" width="50%">
   <img src="capturas/t4-paso-10-detalle-job.png" alt="Detalle job" width="50%">

**Resultado esperado:**
> Diagnostic settings configurado. Los jobs de backup son visibles en el panel de monitoreo.

---

## Tarea 5 — Habilitar replicación con Site Recovery

**Objetivo de la tarea:** Azure Site Recovery replica la VM hacia una región secundaria continuamente. En caso de desastre regional, puedes hacer failover en minutos.

### Método A — Portal web

1. Portal → **Recovery Services vaults** → **+ Create** en **West US**:
   - **RG:** `az104-rg-region2` | **Name:** `az104-rsv-region2`

   <img src="capturas/t5-paso-01-crear-rsv-region2.png" alt="Crear RSV region2" width="50%">
   <img src="capturas/t5-paso-02-rsv-region2-basics.png" alt="RSV region2 basics" width="50%">

2. Portal → busca `az104-10-vm0`

   <img src="capturas/t5-paso-03-buscar-vm0.png" alt="Buscar VM0" width="50%">

3. **Backup + Disaster recovery → Disaster recovery**

   <img src="capturas/t5-paso-04-disaster-recovery-menu.png" alt="Disaster recovery menú" width="50%">

4. **Basics** → verifica la **Target region:** West US

   <img src="capturas/t5-paso-05-target-region.png" alt="Target region" width="50%">

5. **Advanced settings** → revisa las opciones

   <img src="capturas/t5-paso-06-advanced-settings.png" alt="Advanced settings" width="50%">

6. **Review + Start replication** → **Enable replication**

   <img src="capturas/t5-paso-07-enable-replication.png" alt="Enable replication" width="50%">
   <img src="capturas/t5-paso-08-replication-iniciada.png" alt="Replication iniciada" width="50%">

> ⏱️ La replicación inicial tarda 10-15 minutos. Continúa mientras se completa.

7. Navega a `az104-rsv-region2` → **Protected items → Replicated items**

   <img src="capturas/t5-paso-09-rsv-region2-overview.png" alt="RSV region2 overview" width="50%">
   <img src="capturas/t5-paso-10-replicated-items.png" alt="Replicated items" width="50%">

8. Verifica estado **Healthy** y synchronization progress

   <img src="capturas/t5-paso-11-vm-healthy.png" alt="VM healthy" width="50%">
   <img src="capturas/t5-paso-12-replication-progress.png" alt="Replication progress" width="50%">
   <img src="capturas/t5-paso-13-vm-details.png" alt="VM details" width="50%">

**Resultado esperado:**
> La VM `az104-10-vm0` aparece en el RSV de West US con estado **Protected** y synchronization completada.

---

## ⚠️ Errores comunes

### Error 1 — El backup falla con `UserErrorVmNotInDesirableState`

**Síntoma:** El job de backup falla inmediatamente.

**Causa:** La VM está en estado deallocated — Azure Backup requiere que la VM esté encendida para el primer backup.

**Solución:** Inicia la VM antes de ejecutar el backup:
```powershell
az vm start --name "az104-10-vm0" --resource-group "az104-rg-region1"
```

---

### Error 2 — Site Recovery falla con `AutomationAccountNotFound`

**Síntoma:** Error al habilitar la replicación por falta de cuenta de automatización.

**Causa:** La creación de la cuenta de automatización es obligatoria en los **Advanced settings** y a veces no se completa.

**Solución:** En Advanced settings, verifica que la Automation Account está completa antes de hacer clic en Enable replication.

---

## 🧹 Limpieza de recursos

> ⚠️ Para eliminar un RSV con datos hay que eliminar primero todos los items protegidos y deshabilitar soft delete.

### Método A — Portal

1. RSV → **Backup items** → elimina cada item protegido
2. RSV → **Properties → Security → Soft Delete:** deshabilita
3. Luego elimina los Resource Groups:

```powershell
az group delete --name "az104-rg-region1" --yes --no-wait
az group delete --name "az104-rg-region2" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| VM Standard_D2s_v3 | ~1 hora | ~$0.096 |
| Azure Backup (1 VM protegida) | ~1 hora | ~$0.014 |
| Azure Site Recovery | ~1 hora | ~$0.035 |
| Storage Account diagnóstico | ~1 hora | ~$0.001 |
| **Total lab** | | **~$0.15** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US → West US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Backup gestionado | Azure Backup | AWS Backup | Backup and DR | Oracle Backup |
| Disaster Recovery | Azure Site Recovery | DRS (Elastic DR) | Backup and DR | Full Stack DR |
| Costo backup VM/mes | ~$10 + storage | ~$0.05/GB + transfer | ~$0.08/GB | ~$0.06/GB |
| Costo replicación/VM/mes | ~$25 | ~$27 (DRS) | [VERIFICAR COSTO] | ~$20 |

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- Backup y Site Recovery son complementarios: Backup protege contra pérdida de datos, ASR protege contra pérdida de toda una región
- El soft delete de 14 días es la protección final contra eliminaciones accidentales de backups
- Las políticas de retención definen el balance entre cobertura (cuántos puntos de restauración) y costo de almacenamiento

**Cómo se usa en labs futuros:**
> El RSV y los diagnostic settings configurados aquí son la referencia para el **Lab-11 Monitoring**, donde se consultan los logs de backup con Azure Monitor y KQL.

**Siguiente lab recomendado:**
👉 [Lab 11 — Monitoring](../../Lab-11-Monitoring/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
