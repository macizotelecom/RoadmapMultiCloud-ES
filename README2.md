# 🗺️ RoadmapMultiCloud-ES

> **Roadmap completo de certificaciones cloud para ingenieros hispanohablantes**  
> Laboratorios progresivos · Análisis FinOps comparativo · Azure · AWS · GCP · OCI  
> Documentado en español · Licencia CC BY-NC-SA 4.0

[![Licencia: CC BY-NC-SA 4.0](https://img.shields.io/badge/Licencia-CC%20BY--NC--SA%204.0-lightgrey.svg)](LICENSE)
[![Estado](https://img.shields.io/badge/Estado-En%20construcci%C3%B3n-yellow)]()
[![Idioma](https://img.shields.io/badge/Idioma-Espa%C3%B1ol-blue)]()
[![Azure](https://img.shields.io/badge/Azure-AZ--900%20%E2%86%92%20AZ--305-0078D4)]()
[![AWS](https://img.shields.io/badge/AWS-CLF%20%E2%86%92%20SAP-FF9900)]()
[![GCP](https://img.shields.io/badge/GCP-CDL%20%E2%86%92%20PCA-4285F4)]()
[![OCI](https://img.shields.io/badge/OCI-1Z0--1085--25%20%E2%86%92%201Z0--997--25-F80000)]()

---

## 🎯 ¿Qué es este proyecto?

**RoadmapMultiCloud-ES** es un learning path estructurado y progresivo para ingenieros IT
con base cloud que quieren certificarse y especializarse como **Cloud Administrators con
perfil de seguridad y arquitectura**.

No es una lista de recursos. Es un sistema de laboratorios que se construyen uno sobre
otro — cada lab avanzado integra todo lo aprendido antes — con análisis de costos reales
documentados y comparativa FinOps entre los cuatro grandes clouds.

### ¿Para quién es esto?

✅ Ingeniero IT con base cloud que quiere certificarse  
✅ Profesional que ya conoce algo de Azure o AWS y quiere estructurar su camino  
✅ Alguien que quiere entender las diferencias reales entre clouds (no solo teoría)  
❌ No está dirigido a quienes empiezan desde cero en IT  

---

## 🗺️ El Roadmap completo

El proyecto está organizado en **4 fases progresivas**. Cada fase tiene su equivalente
en los 4 clouds principales. Azure es el eje central del aprendizaje.

| Fase | Objetivo | Azure | AWS | GCP | OCI |
|------|----------|-------|-----|-----|-----|
| **F1** | Fundamentos — el lenguaje común | AZ-900 | CLF-C02 | CDL | 1Z0-1085-25 |
| **F2** | Administración — operar en producción | AZ-104 | SAA-C03 | ACE | 1Z0-1093-25 |
| **F3** | Seguridad — hardening y compliance | AZ-500 | SCS-C03 | PCSE | 1Z0-1104-25 |
| **F4** | Arquitectura — diseñar a escala | AZ-305 | SAP-C02 | PCA | 1Z0-997-25 |
| **FinOps** | Comparativa transversal de costos | Azure Cost Mgmt | Cost Explorer | Cost Tools | OCI Cost |

> ⚠️ **Nota:** AWS SCS-C02 fue retirado en diciembre 2025. El examen vigente es **SCS-C03**.

---

## 🧩 Estructura de cada laboratorio

Cada módulo temático tiene **3 niveles de profundidad**:

```
Lab-XX-nombre/
├── lab-base/          # Nivel 1 — Conoce
│   ├── README.md      # Lab oficial enriquecido en español
│   ├── capturas/      # Screenshots de cada tarea
│   └── costos.md      # Costo real documentado con billing
│
├── lab-avanzado/      # Nivel 2 — Aplica
│   ├── README.md      # Escenario de empresa real
│   ├── iac/           # Bicep / CloudFormation / Terraform
│   ├── scripts/       # CLI (az / aws / gcloud / oci)
│   └── troubleshooting.md
│
└── finops/            # Nivel 3 — Decide
    ├── README.md      # Comparativa de costos entre 4 clouds
    ├── tabla-costos.md
    └── cuando-elegir.md
```

---

## 🚀 Estado del proyecto — AZ-104 Labs

| Lab | Título | Capturas | README | Estado |
|-----|--------|----------|--------|--------|
| 01 | Entra ID Identidades | ✅ | ✅ | ✅ Completo |
| 02a | RBAC | ✅ | ✅ | ✅ Completo |
| 02b | Azure Policy | ✅ | ✅ | ✅ Completo |
| 03 | ARM Templates y Bicep | ✅ | ✅ | ✅ Completo |
| 04 | Virtual Networking | ✅ | ✅ | ✅ Completo |
| 05 | Intersite Connectivity | ✅ | ✅ | ✅ Completo |
| 06 | Network Traffic Management | ✅ | ✅ | ✅ Completo |
| 07 | Azure Storage | ✅ | ✅ | ✅ Completo |
| 08 | Virtual Machines | ✅ | ✅ | ✅ Completo |
| 09a | Web Apps PaaS | ✅ | ✅ | ✅ Completo |
| 09b | Container Instances | ✅ | ✅ | ✅ Completo |
| 09c | Container Apps | ✅ | ✅ | ✅ Completo |
| 10 | Data Protection y Backup | ✅ | ✅ | ✅ Completo |
| 11 | Monitoring | ✅ | ✅ | ✅ Completo |

---

## 📋 Labs AZ-104 — Fase 2 Administración

Todos los labs están documentados en español con capturas por cada tarea, 3 métodos
(Portal / CLI / IaC), errores comunes y análisis FinOps comparativo con AWS, GCP y OCI.

### [**Lab 01 — Manage Microsoft Entra ID Identities**](fase-2-administracion/azure/az-104/Lab-01-Entra-ID/lab-base/README.md)
Creación de usuarios, usuarios invitados y grupos en **Microsoft Entra ID**. Configuración de
licencias, propiedades y tipos de miembrosía (asignada vs dinámica).

### [**Lab 02a — Manage Subscriptions and RBAC**](fase-2-administracion/azure/az-104/Lab-02a-RBAC/lab-base/README.md)
Asignación de roles built-in a usuarios y grupos, verificación de permisos efectivos
y creación de **custom roles** con el principio de mínimo privilegio.

### [**Lab 02b — Manage Governance via Azure Policy**](fase-2-administracion/azure/az-104/Lab-02b-Policy/lab-base/README.md)
Asignación de políticas built-in, efectos Audit/Deny/DeployIfNotExists, creación de
**políticas personalizadas** e **Initiatives** para gobernanza de entornos.

### [**Lab 03 — Manage Azure Resources using ARM Templates**](fase-2-administracion/azure/az-104/Lab-03-ARM-Bicep/lab-base/README.md)
Exportar plantillas ARM desde el portal, editar y redesplegar. Despliegue con
**PowerShell**, **CLI** y **Bicep** desde Cloud Shell — cinco discos, cinco métodos.

### [**Lab 04 — Implement Virtual Networking**](fase-2-administracion/azure/az-104/Lab-04-VNets/lab-base/README.md)
Creación de VNets con subredes desde portal y plantilla ARM. Configuración de
**ASG + NSG** para control granular de tráfico. Zonas DNS públic y privada.

### [**Lab 05 — Implement Intersite Connectivity**](fase-2-administracion/azure/az-104/Lab-05-Intersite/lab-base/README.md)
VMs en VNets distintas, diagnóstico con Network Watcher, configuración de
**VNet Peering** bidireccional y rutas personalizadas **UDR** hacia NVA.

### [**Lab 06 — Implement Network Traffic Management**](fase-2-administracion/azure/az-104/Lab-06-Traffic-Manager/lab-base/README.md)
**Azure Load Balancer** (capa 4) con backend pool y health probes.
**Azure Application Gateway** (capa 7) con path-based routing para imágenes y videos.

### [**Lab 07 — Manage Azure Storage**](fase-2-administracion/azure/az-104/Lab-07-Storage/lab-base/README.md)
Cuenta de almacenamiento GRS con lifecycle management. Blob Storage con retención
inmutable y **SAS tokens**. Azure File Share con restricción por service endpoint.

### [**Lab 08 — Manage Virtual Machines**](fase-2-administracion/azure/az-104/Lab-08-VMs/lab-base/README.md)
VMs en **Availability Zones** para SLA 99.99%. Escalado vertical de SKU y discos.
**Virtual Machine Scale Sets** con autoscaling basado en CPU. VMs con PowerShell y CLI.

### [**Lab 09a — Implement Web Apps**](fase-2-administracion/azure/az-104/Lab-09a-PaaS-Web/lab-base/README.md)
Azure App Service con **deployment slots** (staging/producción). Despliegue continuo
desde GitHub y **swap sin downtime**. Autoscaling con prueba de carga.

### [**Lab 09b — Implement Azure Container Instances**](fase-2-administracion/azure/az-104/Lab-09b-ACI/lab-base/README.md)
Despliegue de imagen Docker en **ACI** sin gestionar infraestructura. Configuración
de DNS público y verificación de logs HTTP del contenedor.

### [**Lab 09c — Implement Azure Container Apps**](fase-2-administracion/azure/az-104/Lab-09c-AKS/lab-base/README.md)
Creación de entorno y despliegue en **Azure Container Apps**. Kubernetes administrado
sin gestionar el cluster — escalado a cero incluido.

### [**Lab 10 — Implement Data Protection**](fase-2-administracion/azure/az-104/Lab-10-Backup/lab-base/README.md)
**Recovery Services Vault** con políticas de backup y soft delete. Diagnósticos hacia
Storage Account. Replicación entre regiones con **Azure Site Recovery**.

### [**Lab 11 — Implement Monitoring**](fase-2-administracion/azure/az-104/Lab-11-Monitoring/lab-base/README.md)
**VM Insights** y alertas de Activity Log. Action Groups con notificaciones email.
Reglas de supresión para mantenimiento. Consultas **KQL** en Log Analytics.

---

## 💰 Filosofía de costos

Todos los labs documentan el **costo real incurrido** con capturas del portal de billing.
El objetivo es que puedas planificar tu gasto de estudio con datos reales, no estimados.

Como referencia: los 14 labs del AZ-104 ejecutados completos cuestan aproximadamente
**$8–12 USD** si se limpian los recursos al terminar cada lab.

---

## 📁 Estructura del repositorio

```
RoadmapMultiCloud-ES/
│
├── README.md                        ← Este archivo · índice maestro
├── LICENSE                          ← CC BY-NC-SA 4.0
├── CONTRIBUTING.md                  ← Cómo contribuir
├── TEMPLATE-LAB.md                  ← Plantilla universal de lab
├── ANALISIS-REPOS.md                ← Análisis de referencia y correcciones
│
├── fase-1-fundamentos/
│   ├── azure/az-900/
│   ├── aws/clf-c02/
│   ├── gcp/cdl/
│   └── oci/1z0-1085-25/
│
├── fase-2-administracion/
│   ├── azure/az-104/
│   │   ├── README.md                ← Índice AZ-104 + costos totales
│   │   ├── Lab-01-Entra-ID/         ✅ Completo
│   │   ├── Lab-02a-RBAC/            ✅ Completo
│   │   ├── Lab-02b-Policy/          ✅ Completo
│   │   ├── Lab-03-ARM-Bicep/        ✅ Completo
│   │   ├── Lab-04-VNets/            ✅ Completo
│   │   ├── Lab-05-Intersite/        ✅ Completo
│   │   ├── Lab-06-Traffic-Manager/  ✅ Completo
│   │   ├── Lab-07-Storage/          ✅ Completo
│   │   ├── Lab-08-VMs/              ✅ Completo
│   │   ├── Lab-09a-PaaS-Web/        ✅ Completo
│   │   ├── Lab-09b-ACI/             ✅ Completo
│   │   ├── Lab-09c-AKS/             ✅ Completo
│   │   ├── Lab-10-Backup/           ✅ Completo
│   │   └── Lab-11-Monitoring/       ✅ Completo
│   ├── aws/saa-c03/
│   ├── gcp/ace/
│   └── oci/1z0-1093-25/
│
├── fase-3-seguridad/
│   ├── azure/az-500/
│   ├── aws/scs-c03/                 ← ⚠️ SCS-C02 retirado dic 2025
│   ├── gcp/pcse/
│   └── oci/1z0-1104-25/
│
├── fase-4-arquitectura/
│   ├── azure/az-305/
│   ├── aws/sap-c02/
│   ├── gcp/pca/
│   └── oci/1z0-997-25/
│
└── comparativa-finops/
    ├── README.md
    ├── por-servicio/
    └── tco-scenarios/
```

---

## 🤝 Cómo contribuir

¿Quieres mejorar un lab, añadir capturas actualizadas o traducir contenido?
Lee la [guía de contribución](CONTRIBUTING.md).

Las contribuciones más valiosas son:
- Capturas actualizadas cuando Azure/AWS/GCP cambian su interfaz
- Correcciones de costos reales (los precios cloud cambian frecuentemente)
- Labs equivalentes en AWS, GCP u OCI para módulos que solo tienen Azure
- Traducciones de troubleshooting y errores reales encontrados

---

## ⚠️ Aviso legal

Este proyecto no está afiliado, patrocinado ni respaldado por Microsoft, Amazon Web
Services, Google Cloud ni Oracle Corporation. Los nombres de productos y certificaciones
son marcas registradas de sus respectivos propietarios.

El contenido de los laboratorios está basado en material oficial de Microsoft Learn,
AWS Training, Google Cloud Skills Boost y Oracle Learning, enriquecido con
documentación propia, capturas reales y análisis de costos verificados.

---

## 📄 Licencia

[CC BY-NC-SA 4.0](LICENSE) — Libre para uso educativo con atribución.
Para uso comercial, contacta con los autores.

---

*Construido con ❤️ para la comunidad cloud hispanohablante.*
