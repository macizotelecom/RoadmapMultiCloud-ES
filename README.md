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

| Fase | Objetivo | Azure | AWS | GCP | OCI |
|------|----------|-------|-----|-----|-----|
| **F1** | Fundamentos | AZ-900 | CLF-C02 | CDL | 1Z0-1085-25 |
| **F2** | Administración | AZ-104 | SAA-C03 | ACE | 1Z0-1093-25 |
| **F3** | Seguridad | AZ-500 | SCS-C03 | PCSE | 1Z0-1104-25 |
| **F4** | Arquitectura | AZ-305 | SAP-C02 | PCA | 1Z0-997-25 |

> ⚠️ AWS SCS-C02 fue retirado en diciembre 2025. El examen vigente es **SCS-C03**.

---

## 🧩 Estructura de cada lab

```
Lab-XX-nombre/
├── lab-base/       ← Nivel 1: lab oficial enriquecido + capturas + costo real
├── lab-avanzado/   ← Nivel 2: escenario empresa real + IaC + troubleshooting
└── finops/         ← Nivel 3: comparativa 4 clouds + cuándo elegir cada uno
```

---

## 📋 AZ-104 — Fase 2 Administración

Los 14 labs del AZ-104 están documentados en español con capturas por cada paso,
3 métodos por tarea (Portal / CLI / IaC), errores comunes y análisis FinOps comparativo.
Costo total de los 14 labs ejecutados y limpiados: **~$8–12 USD**.

| # | Lab | Dominio AZ-104 |
|---|-----|---------------|
| 01 | [Entra ID Identidades](fase-2-administracion/azure/az-104/Lab-01-Entra-ID/lab-base/README.md) | Identidades y gobernanza (20–25%) |
| 02a | [RBAC](fase-2-administracion/azure/az-104/Lab-02a-RBAC/lab-base/README.md) | Identidades y gobernanza (20–25%) |
| 02b | [Azure Policy](fase-2-administracion/azure/az-104/Lab-02b-Policy/lab-base/README.md) | Identidades y gobernanza (20–25%) |
| 03 | [ARM Templates y Bicep](fase-2-administracion/azure/az-104/Lab-03-ARM-Bicep/lab-base/README.md) | Recursos Azure (15–20%) |
| 04 | [Virtual Networking](fase-2-administracion/azure/az-104/Lab-04-VNets/lab-base/README.md) | Redes virtuales (25–30%) |
| 05 | [Intersite Connectivity](fase-2-administracion/azure/az-104/Lab-05-Intersite/lab-base/README.md) | Redes virtuales (25–30%) |
| 06 | [Network Traffic Management](fase-2-administracion/azure/az-104/Lab-06-Traffic-Manager/lab-base/README.md) | Redes virtuales (25–30%) |
| 07 | [Azure Storage](fase-2-administracion/azure/az-104/Lab-07-Storage/lab-base/README.md) | Almacenamiento (15–20%) |
| 08 | [Virtual Machines](fase-2-administracion/azure/az-104/Lab-08-VMs/lab-base/README.md) | Compute (20–25%) |
| 09a | [Web Apps PaaS](fase-2-administracion/azure/az-104/Lab-09a-PaaS-Web/lab-base/README.md) | Compute (20–25%) |
| 09b | [Azure Container Instances](fase-2-administracion/azure/az-104/Lab-09b-ACI/lab-base/README.md) | Compute (20–25%) |
| 09c | [Azure Container Apps](fase-2-administracion/azure/az-104/Lab-09c-AKS/lab-base/README.md) | Compute (20–25%) |
| 10 | [Data Protection y Backup](fase-2-administracion/azure/az-104/Lab-10-Backup/lab-base/README.md) | Protección de datos (10–15%) |
| 11 | [Monitoring](fase-2-administracion/azure/az-104/Lab-11-Monitoring/lab-base/README.md) | Monitoreo (10–15%) |

---

## 🚀 Próximas fases

| Fase | Cert | Estado |
|------|------|--------|
| F1 | AZ-900 | 🔜 Próximamente |
| F2 | SAA-C03 (AWS) | 🔜 Próximamente |
| F2 | ACE (GCP) | 🔜 Próximamente |
| F2 | 1Z0-1093-25 (OCI) | 🔜 Próximamente |
| F3 | AZ-500 | 🔜 Próximamente |
| F4 | AZ-305 | 🔜 Próximamente |
| FinOps | Todos los clouds | 🔜 Al finalizar F4 |

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
