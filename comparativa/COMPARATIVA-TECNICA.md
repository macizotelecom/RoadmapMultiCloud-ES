# Comparativa técnica — Los 4 grandes clouds

> Basada exclusivamente en documentación oficial de Microsoft, AWS, Google Cloud y Oracle.
> Última revisión: mayo 2026. Los precios y características cambian — verifica siempre la fuente oficial antes de tomar decisiones.

---

## Por tipo de empresa

| Perfil | Recomendación | Razonamiento |
|--------|--------------|--------------|
| **Startup / MVP** | GCP o AWS | GCP tiene el programa de créditos para startups más generoso. AWS tiene el ecosistema de servicios gestionados más amplio para escalar sin gestión de infraestructura. |
| **Empresa mediana** | Azure o AWS | Azure si el equipo opera con Microsoft 365 y Active Directory — la integración con Entra ID elimina una capa de gestión de identidades. AWS si el stack es agnóstico o predominantemente Linux/open source. |
| **Enterprise** | Azure + AWS (multi-cloud habitual) | Azure lidera en identidades corporativas e integración on-premise. AWS lidera en variedad de servicios gestionados y cobertura de regiones globales. |
| **Sector público / regulado** | Azure o OCI | Azure tiene las certificaciones de compliance más amplias en Europa (ENS, ISO 27001, GDPR). OCI es opción frecuente en banca y telco por su pricing consistente entre regiones y sus contratos soberanos. |

---

## Por perfil técnico

| Perfil | Cloud recomendado | Razonamiento |
|--------|------------------|--------------|
| **Cloud Administrator** | Azure | La integración nativa con Entra ID es la opción más madura para entornos Windows/Microsoft. RBAC y Azure Policy tienen mayor profundidad que los equivalentes IAM para este perfil. |
| **Arquitecto de soluciones** | AWS como referencia | Diseñar en AWS expone al mayor catálogo de servicios gestionados del mercado, lo que amplía el conocimiento de patrones aplicables a cualquier cloud. |
| **DevOps / Platform Engineer** | GCP o AWS | GKE es el servicio Kubernetes con mayor tiempo de madurez en el mercado. AWS tiene el ecosistema CI/CD más completo (CodePipeline, CodeBuild, ECR, EKS). |
| **Data Engineer / ML** | GCP o AWS | BigQuery es el servicio de análisis sin gestión de infraestructura más consolidado. AWS SageMaker tiene el ecosistema MLOps más amplio. |
| **Seguridad / Compliance** | Azure o AWS | Microsoft Sentinel + Defender for Cloud es la plataforma SIEM/SOAR más integrada para entornos Microsoft. AWS Security Hub centraliza findings con alta flexibilidad multi-cuenta. |

---

## Por dominio técnico

### Identidades y acceso

| Cloud | Servicio principal | Punto fuerte | Cuándo elegirlo |
|-------|-------------------|-------------|-----------------|
| **Azure** | Microsoft Entra ID | SSO corporativo nativo, Conditional Access, integración directa con Active Directory on-premise | Entornos con usuarios Windows / Microsoft 365 |
| **AWS** | IAM + IAM Identity Center | Máxima granularidad de permisos, federación SAML 2.0 y OIDC, acceso multi-cuenta | Entornos programáticos, microservicios |
| **GCP** | Cloud IAM + Workload Identity | Identidades para workloads sin claves estáticas, integración profunda con GKE | Entornos Kubernetes-first o cloud-native |
| **OCI** | IAM + Identity Domains | Modelo de compartimentos para aislamiento fuerte de unidades de negocio y tenants | Entornos con múltiples departamentos que requieren separación estricta |

> ✅ **Integración verificada:** Entra ID como IdP para AWS IAM Identity Center vía SAML 2.0 + SCIM. AWS provisiona usuarios y grupos desde Entra ID automáticamente.
> Fuente oficial: [docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html](https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html)

---

### Redes

| Cloud | Característica diferencial | Consideración importante |
|-------|---------------------------|------------------------|
| **Azure** | Hub-and-spoke con Azure Firewall, ExpressRoute, VNet Peering | El peering no es transitivo — se requiere Azure Virtual WAN para topologías de muchas redes |
| **AWS** | Transit Gateway escala a miles de VPCs, Direct Connect maduro | Los rangos CIDR deben planificarse desde el inicio — el solapamiento impide el peering |
| **GCP** | La VPC es un recurso global que abarca todas las regiones. Las subredes sí son regionales | El VPC Peering en GCP tampoco es transitivo, igual que en Azure y AWS |
| **OCI** | FastConnect cobra solo por hora de puerto, no por volumen de datos | Ecosistema de proveedores de conectividad más pequeño que Azure y AWS |

> ✅ **Integración verificada:** Azure + OCI conectados vía ExpressRoute + FastConnect sin costo de egress entre clouds. Requiere SKU local de ExpressRoute, mínimo 1 Gbps, solo en regiones donde ambos coexisten.
> Fuente oficial: [docs.oracle.com](https://docs.oracle.com/en/solutions/oci-best-practices-networking/define-workload-requirements1.html)

---

### Compute

| Cloud | VMs | Contenedores | Serverless | Nota |
|-------|-----|-------------|------------|------|
| **Azure** | VMSS con integración Entra ID nativa | AKS, ACI, Container Apps | Azure Functions, Logic Apps | — |
| **AWS** | EC2 con la mayor variedad de familias de instancias del mercado | EKS, ECS/Fargate | Lambda | — |
| **GCP** | Preemptible VMs y Spot VMs para reducción de costo | GKE | Cloud Run, Cloud Functions | — |
| **OCI** | Ampere A1 ARM: 4 OCPUs y 24 GB RAM en tier gratuito permanente | OKE | Functions | ⚠️ Las instancias con CPU < 20% durante 7 días pueden ser reclamadas por Oracle |

> ℹ️ El tier gratuito permanente de OCI Ampere A1 está documentado en:
> [docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm)

---

### Almacenamiento

| Cloud | Objetos | Archivos | Egress hacia Internet | Punto fuerte |
|-------|---------|---------|----------------------|-------------|
| **Azure** | Blob Storage | Azure Files (SMB/NFS) | Cobrado por GB | Azure File Sync para integración con on-premise |
| **AWS** | S3 | EFS / FSx | 100 GB/mes gratis, luego por GB | S3 es compatible nativamente con la mayoría de herramientas de datos |
| **GCP** | Cloud Storage | Filestore | Precios competitivos por región | Clases de almacenamiento más granulares que S3 |
| **OCI** | Object Storage (compatible con API S3) | File Storage | **10 TB/mes gratis**, luego ~$0.0085/GB | Egress más generoso del mercado |

> ℹ️ El egress gratuito de OCI de 10 TB/mes está documentado en:
> [oracle.com/cloud/networking/virtual-cloud-network/pricing](https://www.oracle.com/cloud/networking/virtual-cloud-network/pricing/)

---

### Seguridad

| Cloud | SIEM / Detección | Gestión de secretos | Punto fuerte |
|-------|-----------------|-------------------|-------------|
| **Azure** | Microsoft Sentinel + Defender for Cloud | Azure Key Vault | Conectores nativos para AWS CloudTrail y GCP Logging — SIEM centralizado multi-cloud |
| **AWS** | Security Hub + GuardDuty + Macie | Secrets Manager + KMS | Mayor flexibilidad para entornos multi-cuenta complejos |
| **GCP** | Security Command Center | Secret Manager + Cloud KMS | Integración con Chronicle (SIEM nativo de Google) |
| **OCI** | Cloud Guard + Security Advisor | Vault + HSM dedicado | HSM dedicado disponible en todos los tiers, no solo enterprise |

> ✅ **Integración verificada:** Microsoft Sentinel ingesta logs de AWS (CloudTrail vía S3 + SQS) y de GCP (Audit Logs, Security Command Center, VPC Flow Logs vía Pub/Sub). Los conectores GCP alcanzaron GA en 2025.
> Fuente oficial: [learn.microsoft.com/en-us/azure/sentinel/connect-aws](https://learn.microsoft.com/en-us/azure/sentinel/connect-aws)

---

## Integraciones multi-cloud verificadas

| Caso de uso | Clouds | Mecanismo | Fuente |
|-------------|--------|-----------|--------|
| Identidad corporativa → acceso AWS | Azure + AWS | Entra ID como IdP → IAM Identity Center vía SAML 2.0 + SCIM | [AWS Docs](https://docs.aws.amazon.com/singlesignon/latest/userguide/idp-microsoft-entra.html) |
| SIEM centralizado multi-cloud | Azure + AWS + GCP | Sentinel con conectores S3/SQS (AWS) y Pub/Sub (GCP) | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/sentinel/connect-aws) |
| Conectividad privada Azure ↔ OCI | Azure + OCI | ExpressRoute + FastConnect sin egress en regiones compartidas | [Oracle Docs](https://docs.oracle.com/en/solutions/oci-best-practices-networking/define-workload-requirements1.html) |
| CI/CD agnóstico multi-cloud | Todos | GitHub Actions con runners y secrets por entorno | [GitHub Docs](https://docs.github.com/en/actions) |
| S3 como hub de datos multi-cloud | AWS + Azure + GCP | Azure Data Factory y GCP Storage Transfer leen S3 de forma nativa | — |

---

> ⚠️ **Nota de mantenimiento:** Los precios, características de tiers gratuitos y disponibilidad de conectores cambian con frecuencia. Verifica siempre contra la documentación oficial antes de tomar decisiones de arquitectura o presupuesto.
> Última revisión: mayo 2026.
