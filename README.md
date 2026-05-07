# 🗺️ RoadmapMultiCloud-ES

> **Roadmap completo de certificaciones cloud para ingenieros hispanohablantes**  
> Laboratorios progresivos · Análisis FinOps comparativo · Azure · AWS · GCP · OCI  
> Documentado en español · Licencia CC BY-NC-SA 4.0

[![Licencia: CC BY-NC-SA 4.0](https://img.shields.io/badge/Licencia-CC%20BY--NC--SA%204.0-lightgrey.svg)](LICENSE)
[![Estado](https://img.shields.io/badge/Estado-En%20construcción-yellow)]()
[![Idioma](https://img.shields.io/badge/Idioma-Español-blue)]()
[![Azure](https://img.shields.io/badge/Azure-AZ--900%20→%20AZ--305-0078D4)]()
[![AWS](https://img.shields.io/badge/AWS-CLF%20→%20SAP-FF9900)]()
[![GCP](https://img.shields.io/badge/GCP-CDL%20→%20PCA-4285F4)]()
[![OCI](https://img.shields.io/badge/OCI-Foundations%20→%20Architect-F80000)]()

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
| **F1** | Fundamentos — el lenguaje común | AZ-900 | CLF-C02 | CDL | 1Z0-1085-25 \|
| **F2** | Administración — operar en producción | AZ-104 | SAA-C03 | ACE | 1Z0-1093-25 \|
| **F3** | Seguridad — hardening y compliance | AZ-500 | SCS-C03 | PCSE | 1Z0-1104-25 |
| **F4** | Arquitectura — diseñar a escala | AZ-305 | SAP-C02 | PCA | 1Z0-997-25 |
| **FinOps** | Comparativa transversal de costos | Azure Cost Mgmt | Cost Explorer | Cost Tools | OCI Cost |

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

### Nivel 1 · Lab base — *Conoce*
Lab oficial enriquecido con capturas por cada tarea, costo real documentado
(screenshot de billing incluido), errores comunes y limpieza de recursos
con 3 métodos: Portal · CLI · IaC.

### Nivel 2 · Lab avanzado — *Aplica*
Escenario de empresa ficticia pero realista que **integra todos los labs anteriores**.
Incluye versión IaC, scripts CLI y sección de troubleshooting real.

### Nivel 3 · Análisis FinOps — *Decide*
Para cada servicio del lab: tabla comparativa con el equivalente en los otros clouds,
costo real por hora de lab, costo mensual proyectado a escala, herramienta nativa de
gestión de costos y recomendación de cuándo elegir cada cloud.

---

## 📁 Estructura del repositorio

```
RoadmapMultiCloud-ES/
│
├── README.md                        ← Este archivo · índice maestro
├── LICENSE                          ← CC BY-NC-SA 4.0
├── CONTRIBUTING.md                  ← Cómo contribuir
├── TEMPLATE-LAB.md                  ← Plantilla universal de lab
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
│   │   ├── Lab-01-Entra-ID/
│   │   ├── Lab-02a-RBAC/
│   │   ├── Lab-02b-Policy/
│   │   ├── Lab-03-ARM-Bicep/
│   │   ├── Lab-04-VNets/
│   │   ├── Lab-05-Intersite/
│   │   ├── Lab-06-Traffic-Manager/
│   │   ├── Lab-07-Storage/
│   │   ├── Lab-08-VMs/
│   │   ├── Lab-09a-PaaS-Web/
│   │   ├── Lab-09b-ACI/
│   │   ├── Lab-09c-AKS/
│   │   ├── Lab-10-Backup/
│   │   └── Lab-11-Monitoring/
│   ├── aws/saa-c03/
│   ├── gcp/ace/
│   └── oci/1z0-1093-25/
│
├── fase-3-seguridad/
│   ├── azure/az-500/
│   ├── aws/SCS-C03/
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
    ├── README.md                    ← Análisis transversal completo
    ├── por-servicio/
    │   ├── identidades.md
    │   ├── redes.md
    │   ├── almacenamiento.md
    │   ├── compute.md
    │   ├── seguridad.md
    │   └── monitoring.md
    └── tco-scenarios/
        ├── startup.md
        ├── empresa-mediana.md
        └── enterprise.md
```

---

## 🚀 Estado del proyecto

| Fase | Cloud | Cert | Labs | Estado |
|------|-------|------|------|--------|
| F1 | Azure | AZ-900 | 0 / 6 | 🔜 Próximamente |
| F2 | Azure | AZ-104 | 0 / 14 | 🔜 Próximamente |
| F2 | AWS | SAA-C03 | 0 / 14 | 🔜 Próximamente |
| F2 | GCP | ACE | 0 / 10 | 🔜 Próximamente |
| F2 | OCI | 1Z0-1093-25 \| 0 / 8 | 🔜 Próximamente |
| F3 | Azure | AZ-500 | 0 / 10 | 🔜 Próximamente |
| F4 | Azure | AZ-305 | 0 / 10 | 🔜 Próximamente |
| — | Todos | FinOps | — | 🔜 Al finalizar F4 |

---

## 💰 Filosofía de costos

Todos los labs documentan el **costo real incurrido** con capturas del portal de billing.
El objetivo es que puedas planificar tu gasto de estudio con datos reales, no estimados.

Como referencia: los 14 labs del AZ-104 ejecutados completos cuestan aproximadamente
**$8–12 USD** si se limpian los recursos al terminar cada lab.

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
