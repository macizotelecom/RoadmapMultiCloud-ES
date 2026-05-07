# Costos — Lab 01 Entra ID

> Fase: F2 | Cert: AZ-104 | Lab: 01 | Tipo: base

---

## Resumen de costo

| Concepto | Duración | Costo |
|----------|----------|-------|
| Microsoft Entra ID Free tier | — | $0.00 |
| Entra ID Premium P2 (si se activa trial) | — | $0.00 (trial gratuito) |
| **Total lab** | | **$0.00** |

---

## Por qué este lab no genera costo

Microsoft Entra ID tiene una capa gratuita (Free tier) que incluye:

- Gestión de usuarios y grupos (sin límite de objetos)
- Autenticación básica y SSO hasta 10 apps
- Informes de seguridad básicos
- Colaboración B2B (invitaciones de usuario guest)

Todo lo que hacemos en este lab — crear usuarios, invitar un guest y crear un grupo
de seguridad con membresía asignada — cae dentro del Free tier.

**La única excepción:** membresía dinámica de grupos requiere **Entra ID Premium P1**
(aproximadamente $6 USD/usuario/mes en pago mensual, precios de referencia Mayo 2026).
Si activas un trial de 30 días de Premium P2 para probar la membresía dinámica,
ese trial no genera cargo.

---

## Captura de billing

[PENDIENTE — añadir captura de Azure Cost Management tras ejecutar el lab]

Ruta esperada: `capturas/billing-lab-01.png`

> Dado que el costo es $0.00, la captura de billing mostrará $0 o el servicio
> simplemente no aparecerá en el desglose. Se documenta igualmente como evidencia
> de que no se generó costo.

---

## Contexto para planificación

Este es el único lab de los 14 del AZ-104 con costo efectivamente nulo.
Los labs con costo real empiezan en Lab-04 (VNets) y Lab-08 (VMs).

Referencia del coste total de los 14 labs completos: **$8-12 USD** con limpieza de recursos.

---

**Fecha de ejecución:** [PENDIENTE]
**Region:** N/A (servicio global)

---

*Parte del proyecto [RoadmapMultiCloud-ES](../../../../../README.md)*