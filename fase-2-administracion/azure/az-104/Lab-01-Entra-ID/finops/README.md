# FinOps — Lab 01 Entra ID: comparativa de identidades entre clouds

> Fase: F2 | Cert: AZ-104 | Lab: 01 | Tipo: finops

---

## Servicios equivalentes por cloud

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Identity Provider cloud nativo | Microsoft Entra ID | IAM Identity Center | Cloud Identity | OCI IAM |
| Directorio de usuarios | Entra ID | IAM + Identity Center | Cloud Identity | OCI IAM Domains |
| Federación con AD on-premises | Entra Connect | AD Connector | Google Cloud Directory Sync | AD Bridge (OCI) |
| Usuarios guest / externos | B2B (Entra ID) | IAM Identity Center (externo) | Cloud Identity (Guest) | OCI IAM Domains |
| Grupos de seguridad | Security Groups | IAM Groups | Google Groups | OCI Groups |
| Membresía dinámica | Entra ID Premium P1 | No nativo | Google Workspace | No nativo |
| MFA nativo | Entra ID Free / Premium | IAM MFA (gratis) | Cloud Identity (gratis) | OCI MFA (gratis) |
| SSO a apps SaaS | Entra ID (hasta 10 apps gratis) | IAM Identity Center | Cloud Identity | OCI IAM |
| Gobierno de acceso / Access Reviews | Entra ID P2 | AWS Access Analyzer | No equivalente directo | OCI Access Governance |

---

## Comparativa de costos (Mayo 2026)

| Tier | Azure Entra ID | AWS IAM | GCP Cloud Identity | OCI IAM |
|------|---------------|---------|-------------------|---------|
| Usuarios/grupos básicos | **Gratis** | **Gratis** | **Gratis** | **Gratis** |
| MFA | Gratis | Gratis | Gratis | Gratis |
| SSO básico | Gratis (10 apps) | Gratis | Gratis | Gratis |
| Membresía dinámica | P1: ~$6/user/mes | No disponible | Workspace: desde $6/user/mes | No disponible |
| Access Reviews / Governance | P2: ~$9/user/mes | Access Analyzer: por uso | No equivalente | Access Governance: [VERIFICAR COSTO] |
| Conditional Access | P1: ~$6/user/mes | IAM Conditions: gratis | Context-Aware Access: Workspace | OCI IAM Conditions: gratis |

> Los precios son orientativos. Verificar siempre en las calculadoras oficiales antes de planificar presupuesto.

---

## Costo real de este lab

| Cloud | Servicio usado | Costo |
|-------|---------------|-------|
| Azure | Entra ID Free tier | $0.00 |
| AWS | No ejecutado | — |
| GCP | No ejecutado | — |
| OCI | No ejecutado | — |

Este lab tiene costo $0.00 en Azure. Es el único lab del AZ-104 con coste nulo.

---

## Análisis: cuándo elegir cada cloud para identidades

### Azure Entra ID
**Elige Azure cuando:**
- La organización ya usa Microsoft 365 — el tenant existe y los usuarios están sincronizados
- Hay Active Directory on-premises — Entra Connect es la solución más madura del mercado
- Se necesita Conditional Access con granularidad alta (por dispositivo, ubicación, riesgo)
- El ecosistema de apps SaaS usa mayoritariamente integración con Azure AD / OIDC

**Puntos débiles:**
- Membresía dinámica y Access Reviews requieren licencias Premium (coste adicional por usuario)
- Precio escala con el número de usuarios — en organizaciones grandes el coste de P1/P2 es significativo

---

### AWS IAM + Identity Center
**Elige AWS cuando:**
- La infraestructura es mayoritariamente AWS y no hay dependencia de Microsoft
- Se necesita control de acceso granular a recursos AWS (IAM Policies son más expresivas que RBAC de Azure para recursos AWS)
- El presupuesto es ajustado — IAM es completamente gratuito

**Puntos débiles:**
- IAM Identity Center no tiene membresía dinámica nativa
- La gestión de usuarios externos (B2B) es menos fluida que Entra ID
- Para SSO a apps SaaS se necesita configuración adicional vs Entra ID que lo tiene integrado

---

### GCP Cloud Identity
**Elige GCP cuando:**
- La organización ya usa Google Workspace — Cloud Identity está integrado
- El equipo de desarrollo usa herramientas Google (Gmail, Drive, Meet) como base
- Se necesita integración con Kubernetes (GKE usa Workload Identity de forma nativa)

**Puntos débiles:**
- Fuera del ecosistema Google, la integración con apps on-premises es más compleja
- No tiene equivalente directo a Access Reviews de Entra ID P2

---

### OCI IAM
**Elige OCI cuando:**
- La carga de trabajo principal son bases de datos Oracle (Autonomous DB, Exadata)
- Se necesita un IdP con coste muy bajo — OCI IAM Domains incluye más funciones en el tier gratuito
- El contrato Oracle existente hace que OCI sea la opción natural por TCO

**Puntos débiles:**
- Ecosistema más reducido de integraciones con apps SaaS de terceros
- Menor comunidad y documentación en español comparado con Azure o AWS

---

## Escenario TCO: TechNova Solutions (80 usuarios)

TechNova tiene 80 empleados. Calcula el coste anual del IdP según el tier necesario:

| Necesidad | Azure | AWS | GCP | OCI |
|-----------|-------|-----|-----|-----|
| Solo usuarios + grupos + MFA | $0/año | $0/año | $0/año | $0/año |
| + Membresía dinámica | ~$5.760/año (P1 × 80) | No disponible | ~$5.760/año (Workspace) | No disponible |
| + Access Reviews + PIM | ~$8.640/año (P2 × 80) | Por uso (bajo) | No equivalente | [VERIFICAR COSTO] |
| + Conditional Access avanzado | incluido en P1 | Gratis (IAM Conditions) | incluido en Workspace | Gratis |

> Para TechNova con 80 usuarios y requerimientos de membresía dinámica,
> Azure P1 (~$5.760/año) y GCP Workspace son equivalentes en precio.
> Si solo se necesitan usuarios, grupos y MFA básico, los cuatro clouds son gratuitos.

---

## Referencias oficiales

| Cloud | Calculadora / Pricing |
|-------|----------------------|
| Azure | [azure.microsoft.com/pricing/details/active-directory](https://azure.microsoft.com/pricing/details/active-directory/) |
| AWS | [aws.amazon.com/iam/pricing](https://aws.amazon.com/iam/pricing/) |
| GCP | [workspace.google.com/pricing](https://workspace.google.com/pricing) |
| OCI | [oracle.com/cloud/price-list](https://www.oracle.com/cloud/price-list/) |

---

*Lab documentado por [RoadmapMultiCloud-ES](../../../../../README.md) · Licencia [CC BY-NC-SA 4.0](../../../../../LICENSE)*