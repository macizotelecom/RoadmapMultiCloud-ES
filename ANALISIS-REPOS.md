# Análisis de repositorios de referencia y recomendaciones

## Repositorio de referencia — Slider2019

**URL:** https://github.com/Slider2019/AZ-104-Microsoft-Azure-Administrator-Associate

### Estructura
```
AZ-104-Microsoft-Azure-Administrator-Associate/
├── README.md
├── LICENSE            ← MIT
└── images/
    ├── Fondoreadme.png
    ├── 1.png          ← captura costo suscripción 1
    └── 2.png          ← captura costo suscripción 2
```

### Modelo de organización
- Cada lab es un **repositorio independiente** en GitHub
- El README principal actúa como **hub de navegación** con links a cada repo de lab
- Cubre Labs 01–11 del AZ-104 (14 labs en total) traducidos al español con capturas por tarea
- Costo total documentado: ~$10 USD en 2 suscripciones

### Lo que adoptamos de su estructura
- README principal como hub con índice, descripción, costos y links
- Carpeta `capturas/` para screenshots de cada tarea y billing
- Documentación del costo real con captura del portal de billing
- Licencia explícita en el repo

### Lo que diferencia nuestro proyecto
- 4 clouds por fase (Azure · AWS · GCP · OCI) en lugar de solo Azure
- 3 tipos de lab por módulo: base / avanzado / FinOps
- Cabecera estandarizada obligatoria en cada entregable
- Licencia CC BY-NC-SA 4.0 en lugar de MIT
- Estructura de carpetas interna (no repos independientes) — más fácil de mantener a escala multi-cloud

---

## Repositorio propio — macizotelecom

**URL:** https://github.com/macizotelecom/RoadmapMultiCloud-ES

### Estructura actual
```
RoadmapMultiCloud-ES/
├── README.md                  ← índice maestro completo ✓
├── LICENSE                    ← CC BY-NC-SA 4.0 ✓
├── CONTRIBUTING.md            ← guía de contribución ✓
├── TEMPLATE-LAB.md            ← plantilla universal de lab ✓
├── ANALISIS-REPOS.md          ← este archivo ✓
├── fase-1-fundamentos/
├── fase-2-administracion/
│   └── azure/az-104/          ← índice + estructura de 14 labs pendiente
├── fase-3-seguridad/
├── fase-4-arquitectura/
└── comparativa-finops/
```

---

## Correcciones aplicadas (Mayo 2026)

### Códigos de certificación actualizados

| Nivel | Azure | AWS | GCP | OCI |
|-------|-------|-----|-----|-----|
| Fundamentos | AZ-900 | CLF-C02 | CDL | 1Z0-1085-25 |
| Administración | AZ-104 | SAA-C03 | ACE | 1Z0-1093-25 |
| Seguridad | AZ-500 | SCS-C03 ✅ | PCSE | 1Z0-1104-25 |
| Arquitectura | AZ-305 | SAP-C02 | PCA | 1Z0-997-25 |

**Cambios aplicados en este commit:**
- `SCS-C02` → `SCS-C03` (SCS-C02 retirado en diciembre 2025)
- Códigos OCI actualizados de nombres genéricos a códigos `1Z0-XXXX-25`
- Badge OCI en README actualizado con códigos 1Z0
- Árbol de carpetas: `aws/scs-c02/` → `aws/scs-c03/`
- Árbol de carpetas: carpetas OCI renombradas con códigos completos
- Tabla de estado: añadida fila F3/AWS SCS-C03

---

## Plan de trabajo — Próximos pasos

| Orden | Entregable | Ruta destino | Estado |
|-------|-----------|--------------|--------|
| 1 | README.md principal | `/README.md` | ✅ Completado |
| 2 | ANALISIS-REPOS.md | `/ANALISIS-REPOS.md` | ✅ Completado |
| 3 | Índice AZ-104 | `/fase-2-administracion/azure/az-104/README.md` | 🔜 Siguiente |
| 4 | Lab-01-Entra-ID base | `/fase-2-administracion/azure/az-104/Lab-01-Entra-ID/lab-base/README.md` | 🔜 Pendiente |
| 5 | Lab-02a-RBAC base | `/fase-2-administracion/azure/az-104/Lab-02a-RBAC/lab-base/README.md` | 🔜 Pendiente |
| 6 | Lab-02b-Policy base | `/fase-2-administracion/azure/az-104/Lab-02b-Policy/lab-base/README.md` | 🔜 Pendiente |
| 7 | Lab-03-ARM-Bicep base | `/fase-2-administracion/azure/az-104/Lab-03-ARM-Bicep/lab-base/README.md` | 🔜 Pendiente |
| 8 | Lab-04-VNets base | `/fase-2-administracion/azure/az-104/Lab-04-VNets/lab-base/README.md` | 🔜 Pendiente |
| 9 | Lab-05-Intersite base | `/fase-2-administracion/azure/az-104/Lab-05-Intersite/lab-base/README.md` | 🔜 Pendiente |
| 10 | Lab-06-Traffic-Manager base | `/fase-2-administracion/azure/az-104/Lab-06-Traffic-Manager/lab-base/README.md` | 🔜 Pendiente |
| 11 | Lab-07-Storage base | `/fase-2-administracion/azure/az-104/Lab-07-Storage/lab-base/README.md` | 🔜 Pendiente |
| 12 | Lab-08-VMs base | `/fase-2-administracion/azure/az-104/Lab-08-VMs/lab-base/README.md` | 🔜 Pendiente |
| 13 | Lab-09a-PaaS-Web base | `/fase-2-administracion/azure/az-104/Lab-09a-PaaS-Web/lab-base/README.md` | 🔜 Pendiente |
| 14 | Lab-09b-ACI base | `/fase-2-administracion/azure/az-104/Lab-09b-ACI/lab-base/README.md` | 🔜 Pendiente |
| 15 | Lab-09c-AKS base | `/fase-2-administracion/azure/az-104/Lab-09c-AKS/lab-base/README.md` | 🔜 Pendiente |
| 16 | Lab-10-Backup base | `/fase-2-administracion/azure/az-104/Lab-10-Backup/lab-base/README.md` | 🔜 Pendiente |
| 17 | Lab-11-Monitoring base | `/fase-2-administracion/azure/az-104/Lab-11-Monitoring/lab-base/README.md` | 🔜 Pendiente |

---

*Análisis realizado: Mayo 2026*  
*Contexto: proyecto RoadmapMultiCloud-ES — macizotelecom*
