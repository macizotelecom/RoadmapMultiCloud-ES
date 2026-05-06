# AZ-104: Microsoft Azure Administrator — Índice de Laboratorios

> Fase: F2 | Cert: AZ-104 | Tipo: índice  
> **Referencia oficial:** [Microsoft Learn — AZ-104](https://microsoftlearning.github.io/AZ-104-MicrosoftAzureAdministrator/)  
> **Última actualización:** 2026-05

---

## 📋 Descripción

Esta sección cubre los **14 laboratorios del AZ-104** traducidos al español y enriquecidos
con el formato del proyecto RoadmapMultiCloud-ES: 3 métodos por tarea (Portal/CLI/IaC),
captura de billing real, análisis FinOps comparativo y escenarios de empresa.

Cada lab tiene 3 niveles: **base** (conoce) → **avanzado** (aplica) → **finops** (decide).

Los labs base están basados en el trabajo de
[Slider2019/AZ-104-Microsoft-Azure-Administrator-Associate](https://github.com/Slider2019/AZ-104-Microsoft-Azure-Administrator-Associate),
enriquecidos con IaC, análisis FinOps multi-cloud y escenarios de empresa reales.

---

## 🗺️ Mapa de laboratorios

| Lab | Nombre | Dominio AZ-104 | Base | Avanzado | FinOps |
|-----|--------|----------------|------|----------|--------|
| 01 | [Entra ID Identidades](Lab-01-Entra-ID/) | Identidades y gobernanza (20–25%) | 🔜 | 🔜 | 🔜 |
| 02a | [RBAC](Lab-02a-RBAC/) | Identidades y gobernanza (20–25%) | 🔜 | 🔜 | 🔜 |
| 02b | [Azure Policy](Lab-02b-Policy/) | Identidades y gobernanza (20–25%) | 🔜 | 🔜 | 🔜 |
| 03 | [ARM Templates / Bicep](Lab-03-ARM-Bicep/) | Recursos Azure (15–20%) | 🔜 | 🔜 | 🔜 |
| 04 | [Virtual Networking](Lab-04-VNets/) | Redes virtuales (25–30%) | 🔜 | 🔜 | 🔜 |
| 05 | [Intersite Connectivity](Lab-05-Intersite/) | Redes virtuales (25–30%) | 🔜 | 🔜 | 🔜 |
| 06 | [Traffic Manager](Lab-06-Traffic-Manager/) | Redes virtuales (25–30%) | 🔜 | 🔜 | 🔜 |
| 07 | [Azure Storage](Lab-07-Storage/) | Almacenamiento (15–20%) | 🔜 | 🔜 | 🔜 |
| 08 | [Virtual Machines](Lab-08-VMs/) | Compute (20–25%) | 🔜 | 🔜 | 🔜 |
| 09a | [Web Apps / PaaS](Lab-09a-PaaS-Web/) | Compute (20–25%) | 🔜 | 🔜 | 🔜 |
| 09b | [Azure Container Instances](Lab-09b-ACI/) | Compute (20–25%) | 🔜 | 🔜 | 🔜 |
| 09c | [Azure Container Apps](Lab-09c-AKS/) | Compute (20–25%) | 🔜 | 🔜 | 🔜 |
| 10 | [Data Protection / Backup](Lab-10-Backup/) | Protección de datos (10–15%) | 🔜 | 🔜 | 🔜 |
| 11 | [Monitoring](Lab-11-Monitoring/) | Monitoreo (10–15%) | 🔜 | 🔜 | 🔜 |

---

## 💰 Costo total estimado

Ejecutar los 14 labs completos con limpieza de recursos al terminar cada uno: **$8–12 USD**.  
Referencia documentada: ~$10 USD en 2 suscripciones (fuente: Slider2019, validado 2024).

> El costo real de cada lab se documenta en su carpeta `lab-base/costos.md` con captura de billing.

---

## 🧩 Estructura de cada lab

```
Lab-XX-nombre/
├── lab-base/
│   ├── README.md          ← Lab enriquecido en español (Portal + CLI + IaC)
│   ├── capturas/          ← Screenshots de cada tarea + billing
│   └── costos.md          ← Costo real documentado
│
├── lab-avanzado/
│   ├── README.md          ← Escenario empresa real integrando labs anteriores
│   ├── iac/               ← Bicep / Terraform
│   ├── scripts/           ← CLI az
│   └── troubleshooting.md
│
└── finops/
    ├── README.md          ← Comparativa Azure vs AWS vs GCP vs OCI
    ├── tabla-costos.md
    └── cuando-elegir.md
```

---

## 📄 Licencia

[CC BY-NC-SA 4.0](../../../../LICENSE) — Libre para uso educativo con atribución.

---

*Parte del proyecto [RoadmapMultiCloud-ES](../../../../README.md)*
