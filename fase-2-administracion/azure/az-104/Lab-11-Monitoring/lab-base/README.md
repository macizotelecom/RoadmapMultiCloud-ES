# Azure Monitor y Alertas — Lab 11

> Fase: F2 | Cert: AZ-104 | Lab: 11 | Tipo: base

---
**Fase:** F2 — Administración
**Certificación:** AZ-104
**Lab número:** 11
**Tipo:** base
**Última actualización:** 2026-05
**Costo real documentado:** ~$0.15 USD/hora (VM + Log Analytics + alertas)
**Región usada:** East US
**Duración estimada:** 40 minutos

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Provisionar infraestructura y configurar VM Insights](#tarea-1)
- [Tarea 2 — Crear alerta de eliminación de VM](#tarea-2)
- [Tarea 3 — Configurar Action Group con notificación por email](#tarea-3)
- [Tarea 4 — Disparar y validar la alerta](#tarea-4)
- [Tarea 5 — Regla de procesamiento de alertas para mantenimiento](#tarea-5)
- [Tarea 6 — Consultas KQL en Azure Monitor Logs](#tarea-6)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Habilitar VM Insights para monitoreo avanzado de una máquina virtual
- [ ] Crear alertas basadas en señales de actividad (eliminación de VM)
- [ ] Configurar Action Groups para notificaciones por email automáticas
- [ ] Crear reglas de procesamiento para suprimir alertas durante mantenimiento
- [ ] Ejecutar consultas KQL en Log Analytics para analizar métricas de CPU

**Habilidades del examen que cubre este lab:**
> **Monitor and back up Azure resources (10–15%)**
> — Configure Azure Monitor alerts
> — Configure Log Analytics
> — Configure Azure Monitor Logs queries

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 03 — ARM/Bicep *(despliegue de plantillas)*

**Conocimientos previos:**
- Azure Monitor centraliza métricas, logs y alertas de todos los recursos Azure
- Action Groups: definen QUÉ hacer cuando se dispara una alerta (email, SMS, webhook, runbook)
- KQL (Kusto Query Language): lenguaje de consulta de Log Analytics — similar a SQL pero optimizado para series temporales

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| VM Standard_D2s_v3 | Recurso a monitorear | ~$0.096/hora |
| Log Analytics Workspace | Almacén de logs y métricas | ~$0.10/GB ingestado |
| Azure Monitor Alerts | Motor de alertas | ~$0.01/regla/mes |
| Action Group | Notificaciones por email | $0.00 (email gratis) |

---

## 🏗️ Arquitectura del lab

```
Azure Monitor
│
├── VM: az104-11-vm0 (East US)
│   └── VM Insights habilitado → Log Analytics Workspace
│
├── Alert Rule: VM was deleted
│   ├── Scope: Suscripción
│   ├── Signal: Delete Virtual Machine (Activity Log)
│   └── Action Group: Alert the operations team
│       └── Notification: Email → tu@correo.com
│
├── Alert Processing Rule: Planned Maintenance
│   ├── Scope: Suscripción
│   ├── Schedule: 10 PM hoy → 7 AM mañana
│   └── Action: Suppress notifications
│
└── Log Analytics Workspace
    └── KQL Query: InsightsMetrics (CPU UtilizationPercentage)
```

---

## Tarea 1 — Provisionar infraestructura y configurar VM Insights

**Objetivo de la tarea:** Desplegar la VM con plantilla y habilitar VM Insights para recopilar métricas avanzadas en Log Analytics.

### Método A — Portal web

1. Descarga `az104-11-vm-template.json` desde la web oficial del lab

   ![Descarga lab11](capturas/t1-paso-01-descarga-lab11.png)

2. Portal → **Deploy a custom template** → **Build your own template**

   ![Custom template](capturas/t1-paso-02-custom-template.png)
   ![Build template](capturas/t1-paso-03-build-template.png)
   ![Load file](capturas/t1-paso-04-load-file.png)

3. Configura: **RG:** `az104-rg11` | **Region:** East US | contraseña → **Create**

   ![Deployment config](capturas/t1-paso-05-deployment-config.png)
   ![Recursos rg11](capturas/t1-paso-06-recursos-rg11.png)
   ![VM desplegada](capturas/t1-paso-07-vm-desplegada.png)

4. Portal → **Monitor** → **VM Insights** → **Configure Insights**

   ![Buscar Monitor](capturas/t1-paso-08-buscar-monitor.png)
   ![VM Insights](capturas/t1-paso-09-vm-insights.png)
   ![Configure Insights](capturas/t1-paso-10-configure-insights.png)

5. **Enable** junto a `az104-11-vm0` → **Review + enable** → **Enable**

   ![Enable VM](capturas/t1-paso-11-enable-vm.png)
   ![Review enable](capturas/t1-paso-12-review-enable.png)
   ![Insights habilitado](capturas/t1-paso-13-insights-habilitado.png)

> ⏱️ La instalación del agente tarda ~5 minutos.

**Resultado esperado:**
> VM Insights habilitado en `az104-11-vm0`. Los datos de CPU, memoria y red empezarán a fluir hacia Log Analytics en minutos.

---

## Tarea 2 — Crear alerta de eliminación de VM

**Objetivo de la tarea:** Crear una regla de alerta que detecte cuando cualquier VM de la suscripción es eliminada. Las alertas basadas en Activity Log detectan operaciones de control (quién creó/eliminó qué) en lugar de métricas de rendimiento.

### Método A — Portal web

1. **Monitor → Alerts** → **+ Create → Alert rule**

   ![Monitor alerts](capturas/t2-paso-01-monitor-alerts.png)
   ![Create alert rule](capturas/t2-paso-02-create-alert-rule.png)

2. **Scope** → marca tu suscripción → **Apply**

   ![Seleccionar suscripción](capturas/t2-paso-03-seleccionar-suscripcion.png)

3. **Condition** → **See all signals**

   ![See all signals](capturas/t2-paso-04-see-all-signals.png)

4. Busca y selecciona **Delete Virtual Machine (Virtual Machines)** → **Apply**

   ![Delete VM signal](capturas/t2-paso-05-delete-vm-signal.png)

5. En **Alert logic** → **Event level:** All selected | **Status:** All selected

   ![Alert logic](capturas/t2-paso-06-alert-logic.png)

**Resultado esperado:**
> Condición de alerta configurada. La ventana de creación permanece abierta para la siguiente tarea.

---

## Tarea 3 — Configurar Action Group con notificación por email

**Objetivo de la tarea:** Los Action Groups definen qué ocurre cuando se dispara una alerta. Un mismo Action Group puede reutilizarse en múltiples reglas de alerta.

### Método A — Portal web

1. Pestaña **Actions** → **Use action groups** → **+ Create action group**

   ![Use action groups](capturas/t3-paso-01-use-action-groups.png)
   ![Create action group](capturas/t3-paso-02-create-action-group.png)

2. **Basics**:
   - **RG:** `az104-rg11` | **Name:** `Alert the operations team` | **Display:** `AlertOpsTeam`

3. **Notifications** → **+ Email/SMS/Push/Voice** → introduce tu email → **OK**

   ![Email notificación](capturas/t3-paso-03-email-notificacion.png)

4. **Review + create** → **Create**

   ![Action group review](capturas/t3-paso-04-action-group-review.png)

5. De vuelta en la alerta → **Details**:
   - **Alert rule name:** `VM was deleted`
   - **Description:** `A VM in your resource group was deleted`

   ![Alert details](capturas/t3-paso-05-alert-details.png)

6. **Review + create** → **Create**

   ![Alert review create](capturas/t3-paso-06-alert-review-create.png)

> 💡 Recibirás un email de bienvenida al Action Group confirmando que está operativo.

**Resultado esperado:**
> Regla de alerta `VM was deleted` creada y vinculada al Action Group. Al eliminar cualquier VM de la suscripción, recibirás un email.

---

## Tarea 4 — Disparar y validar la alerta

**Objetivo de la tarea:** Verificar que la alerta funciona eliminando la VM del lab y confirmando que se recibe la notificación por email.

### Método A — Portal web

1. Portal → **Virtual machines** → selecciona `az104-11-vm0`

   ![Buscar VMs](capturas/t4-paso-01-buscar-vms.png)
   ![Seleccionar vm0](capturas/t4-paso-02-seleccionar-vm0.png)

2. **Delete** → marca **Apply force delete** → confirma y elimina

   ![Delete VM](capturas/t4-paso-03-delete-vm.png)

3. Monitorea la notificación de eliminación completada

   ![Notificación VM deleted](capturas/t4-paso-04-notificacion-vm-deleted.png)

4. Revisa el email del Action Group

   ![Email action group](capturas/t4-paso-05-email-action-group.png)
   ![Email alerta recibido](capturas/t4-paso-06-email-alerta-recibido.png)

5. **Monitor → Alerts** → verifica las alertas generadas

   ![Alertas generadas](capturas/t4-paso-07-alertas-generadas.png)
   ![Alert details](capturas/t4-paso-08-alert-details.png)

**Resultado esperado:**
> Recibes un email con asunto "Azure Monitor alert VM was deleted was activated". En el portal aparecen 3 alertas relacionadas con la eliminación de la VM.

---

## Tarea 5 — Regla de procesamiento de alertas para mantenimiento

**Objetivo de la tarea:** Durante ventanas de mantenimiento programado, las alertas siguen disparándose pero no envían notificaciones. Las reglas de procesamiento gestionan este comportamiento sin modificar las reglas de alerta originales.

### Método A — Portal web

1. **Monitor → Alerts → Alert processing rules** → **+ Create**

   ![Alert processing rules](capturas/t5-paso-01-alert-processing-rules.png)

2. **Scope** → selecciona tu suscripción → **Apply**

   ![Seleccionar suscripción](capturas/t5-paso-02-seleccionar-suscripcion.png)

3. **Rule settings** → **Suppress notifications**

   ![Suppress notifications](capturas/t5-paso-03-suppress-notifications.png)

4. **Scheduling** → **At a specific time**:
   - **Start:** hoy a las 10 PM | **End:** mañana a las 7 AM | **Timezone:** tu zona horaria

5. **Details**:
   - **RG:** `az104-rg11` | **Name:** `Planned Maintenance`
   - **Description:** `Suppress notifications during planned maintenance`

   ![Rule details](capturas/t5-paso-04-rule-details.png)

6. **Review + create** → **Create**

   ![Rule review](capturas/t5-paso-05-rule-review.png)

**Resultado esperado:**
> Regla de supresión creada. Durante la ventana configurada, las alertas se registran pero no envían notificaciones.

---

## Tarea 6 — Consultas KQL en Azure Monitor Logs

**Objetivo de la tarea:** Usar el lenguaje KQL para analizar datos de VM Insights. KQL es el lenguaje nativo de Log Analytics y permite correlacionar datos de múltiples fuentes en tiempo real.

### Método A — Portal web

1. **Monitor → Logs** → cierra el splash screen si aparece

   ![Monitor logs](capturas/t6-paso-01-monitor-logs.png)
   ![Cerrar splash](capturas/t6-paso-02-cerrar-splash.png)

2. Si es necesario, selecciona el **Scope** de tu suscripción

   ![Seleccionar scope](capturas/t6-paso-03-seleccionar-scope.png)

3. Pega y ejecuta la consulta KQL:

```kql
InsightsMetrics
| where TimeGenerated > ago(1h)
| where Name == "UtilizationPercentage"
| summarize avg(Val) by bin(TimeGenerated, 5m), Computer
| render timechart
```

   ![KQL query](capturas/t6-paso-04-kql-query.png)
   ![Gráfico CPU](capturas/t6-paso-05-grafico-cpu.png)

> Si los datos de VM Insights no han llegado todavía (requiere ~15-30 min), el gráfico aparecerá vacío — esto es normal.

### Método B — Consultas adicionales útiles

```kql
-- Heartbeats de la VM (verifica que el agente está activo)
Heartbeat
| where TimeGenerated > ago(1h)
| summarize count() by Computer, bin(TimeGenerated, 5m)

-- Eventos del Activity Log de la suscripción
AzureActivity
| where TimeGenerated > ago(1h)
| where OperationNameValue contains "delete"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue
```

**Resultado esperado:**
> El gráfico de timechart muestra el porcentaje de CPU de la VM en intervalos de 5 minutos durante la última hora.

---

## ⚠️ Errores comunes

### Error 1 — No llegan alertas al email

**Síntoma:** La VM se eliminó pero no recibiste el email.

**Causa:** El email puede tardar 5-10 minutos. También puede estar en la carpeta de spam o haber una regla de procesamiento activa que lo suprime.

**Solución:** Verifica en **Monitor → Alerts** que la alerta existe aunque no haya notificación. Revisa carpeta de spam. Verifica que no tienes reglas de procesamiento activas.

---

### Error 2 — La consulta KQL devuelve tabla vacía

**Síntoma:** La consulta InsightsMetrics no devuelve datos.

**Causa:** VM Insights tarda 15-30 minutos en empezar a ingestar datos. Si la VM fue eliminada antes, no habrá datos.

**Solución:** Usa el [Log Analytics Demo Environment](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade) de Microsoft para practicar KQL con datos reales de ejemplo.

---

## 🧹 Limpieza de recursos

### Método A — Portal

`az104-rg11` → **Eliminar grupo de recursos**

![Limpieza](capturas/limpieza-eliminar-rg.png)

### Método B — CLI

```powershell
az group delete --name "az104-rg11" --yes --no-wait
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| VM Standard_D2s_v3 | ~1 hora | ~$0.096 |
| Log Analytics (datos mínimos) | ~1 hora | ~$0.015 |
| Alert rules (2) | ~1 hora | ~$0.001 |
| **Total lab** | | **~$0.11** |

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]
**Región:** East US

---

## 📊 Análisis FinOps rápido

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Monitoreo centralizado | Azure Monitor | CloudWatch | Cloud Monitoring | OCI Monitoring |
| Logs y consultas | Log Analytics + KQL | CloudWatch Logs + Insights | Cloud Logging + MQL | Logging Analytics |
| Alertas | Monitor Alerts | CloudWatch Alarms | Cloud Alerting | OCI Alarms |
| Action Groups | Action Groups | SNS Topics | Notification Channels | OCI Notifications |
| Costo logs/GB ingestado | $0.10 | $0.50 | $0.01 | $0.01 |

**¿Cuándo usar Azure Monitor vs terceros (Datadog, Grafana)?**
> Azure Monitor es la opción correcta cuando todos los recursos están en Azure — integración nativa, sin agentes adicionales para la mayoría de servicios. Datadog o Grafana tienen más sentido en entornos multi-cloud o cuando el equipo ya domina esas herramientas.

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- Las alertas de Activity Log detectan operaciones (quién hizo qué) — las de métricas detectan rendimiento (cómo está funcionando)
- Los Action Groups son reutilizables: un grupo puede recibir alertas de múltiples reglas
- Las reglas de procesamiento de alertas son el mecanismo correcto para mantenimiento — no modifican las reglas de alerta originales
- KQL es un lenguaje poderoso para análisis de logs — `summarize`, `where`, `render timechart` son las operaciones más comunes

**Cómo se usa en labs futuros:**
> Este es el último lab de la fase F2 AZ-104. Los conceptos de monitoreo se profundizan en la **Fase F3 AZ-500** con Microsoft Defender for Cloud y Sentinel.

**Siguiente fase:**
👉 [AZ-500 — Seguridad](../../../../fase-3-seguridad/azure/az-500/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
