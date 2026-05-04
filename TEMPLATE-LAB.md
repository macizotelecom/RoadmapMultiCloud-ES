# [NOMBRE DEL LAB] — Lab XX

---
**Fase:** F[1|2|3|4] — [Fundamentos|Administración|Seguridad|Arquitectura]  
**Certificación:** [AZ-900|AZ-104|AZ-500|AZ-305|SAA-C03|ACE|...]  
**Lab número:** XX  
**Tipo:** [base|avanzado|finops]  
**Última actualización:** YYYY-MM  
**Costo real documentado:** $X.XX USD  
**Región usada:** [East US|eu-west-1|europe-west1|...]  
**Duración estimada:** XX minutos  

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Nombre](#tarea-1)
- [Tarea 2 — Nombre](#tarea-2)
- [Tarea 3 — Nombre](#tarea-3)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Objetivo 1
- [ ] Objetivo 2
- [ ] Objetivo 3

**Habilidades del examen que cubre este lab:**
> Indicar aquí los dominios y habilidades del examen que cubre este lab.
> Ejemplo: "Manage Azure identities and governance (20–25%)"

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab XX — [Nombre] *(si aplica)*
- [ ] Lab XX — [Nombre] *(si aplica)*

**Acceso necesario:**
- Suscripción Azure activa / Cuenta AWS / Proyecto GCP / Tenancy OCI
- Rol mínimo requerido: [Contributor | Admin | Owner]
- Herramientas instaladas: [Azure CLI | AWS CLI | gcloud | oci cli]

**Conocimientos previos:**
- Concepto 1
- Concepto 2

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| [Servicio 1] | Descripción | $X.XX/hora |
| [Servicio 2] | Descripción | $X.XX/hora |

---

## 🏗️ Arquitectura del lab

```
[Diagrama ASCII o descripción de la arquitectura que se construye]

Ejemplo:
┌─────────────────────────────────────┐
│           Resource Group            │
│  ┌──────────┐    ┌──────────────┐   │
│  │  VNet    │───▶│   Storage    │   │
│  └──────────┘    └──────────────┘   │
└─────────────────────────────────────┘
```

**Lo que construimos en este lab:**
Descripción en 2-3 líneas de qué infraestructura queda desplegada al final.

**Cómo conecta con labs anteriores:**
> Si este lab usa recursos de labs anteriores, indicarlo aquí.

---

## Tarea 1 — [Nombre descriptivo]

**Objetivo de la tarea:** Qué se logra al completar esta tarea.

### Método A — Portal web

1. Navega a [URL o servicio] en el portal
2. Haz clic en **+ Crear** / **+ New**

   ![Captura paso 2](capturas/tarea1-paso-02-nombre.png)

3. Rellena los campos:
   - **Nombre:** `valor-ejemplo`
   - **Región:** `East US`
   - **SKU:** `Standard`

   ![Captura paso 3](capturas/tarea1-paso-03-nombre.png)

4. Haz clic en **Revisar y crear** → **Crear**

   > ⏱️ La operación tarda aproximadamente X minutos.

   ![Captura resultado](capturas/tarea1-paso-04-resultado.png)

### Método B — CLI

```bash
# Azure CLI
az [comando] \
  --name "nombre-recurso" \
  --resource-group "rg-lab-XX" \
  --location "eastus" \
  --sku "Standard"

# AWS CLI
aws [servicio] [comando] \
  --nombre "valor" \
  --region "us-east-1"

# gcloud
gcloud [servicio] [comando] \
  --nombre="valor" \
  --region="us-east1"
```

### Método C — IaC (Bicep / Terraform / CloudFormation)

```bicep
// Azure — Bicep
resource nombre 'Microsoft.Tipo/subtipo@2024-01-01' = {
  name: 'nombre-recurso'
  location: resourceGroup().location
  properties: {
    // propiedades
  }
}
```

```terraform
# Terraform (multi-cloud)
resource "tipo_recurso" "nombre" {
  name     = "nombre-recurso"
  location = "East US"
}
```

**Resultado esperado:**
> Describe qué debería verse o poder verificarse al completar esta tarea correctamente.

---

## Tarea 2 — [Nombre descriptivo]

> *Repite la estructura de Tarea 1 para cada tarea adicional*

---

## Tarea 3 — [Nombre descriptivo]

> *Repite la estructura de Tarea 1 para cada tarea adicional*

---

## ⚠️ Errores comunes

### Error 1 — [Nombre del error o mensaje]

**Síntoma:**
```
Mensaje de error exacto que aparece
```

**Causa:** Explicación de por qué ocurre.

**Solución:**
```bash
# Comando o pasos para resolverlo
```

---

### Error 2 — [Nombre del error]

**Síntoma:** Descripción del síntoma.

**Causa:** Explicación.

**Solución:** Pasos para resolver.

---

## 🧹 Limpieza de recursos

> ⚠️ **Importante:** Elimina los recursos al terminar para evitar costos innecesarios.

### Método A — Portal

1. Navega al Resource Group / consola correspondiente
2. Selecciona los recursos creados en este lab
3. Haz clic en **Eliminar** y confirma

### Método B — CLI

```bash
# Azure — eliminar resource group completo
az group delete --name "rg-lab-XX" --yes --no-wait

# AWS — eliminar recursos específicos
aws [servicio] delete-[recurso] --nombre "valor"

# GCP — eliminar proyecto o recursos
gcloud [servicio] delete [recurso]
```

### Método C — Script automatizado

```bash
#!/bin/bash
# cleanup-lab-XX.sh
# Elimina todos los recursos creados en este lab

echo "Iniciando limpieza del Lab XX..."

# Lista de recursos a eliminar
az group delete --name "rg-lab-XX" --yes --no-wait

echo "Limpieza completada."
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| [Servicio 1] | X horas | $X.XX |
| [Servicio 2] | X horas | $X.XX |
| **Total lab** | | **$X.XX** |

**Captura del billing:**

![Billing del lab](capturas/billing-lab-XX.png)

**Fecha de ejecución:** YYYY-MM-DD  
**Región:** [Región usada]  
**Notas de costo:**
> Si hay algo relevante sobre el costo (pico inesperado, recurso olvidado, etc.)

---

## 📊 Análisis FinOps rápido

*Para el análisis FinOps completo ver: [finops/README.md](../../finops/README.md)*

| Servicio equivalente | Azure | AWS | GCP | OCI |
|---------------------|-------|-----|-----|-----|
| [Tipo de servicio] | [Nombre] | [Nombre] | [Nombre] | [Nombre] |
| Costo estimado/mes | $X | $X | $X | $X |

**¿Cuándo elegir Azure para este caso de uso?**
> Breve respuesta en 2-3 líneas.

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- Punto clave 1
- Punto clave 2
- Punto clave 3

**Cómo se usa en labs futuros:**
> Este lab crea [recurso X] que será usado en el Lab YY para [propósito].

**Siguiente lab recomendado:**
👉 [Lab XX+1 — Nombre del siguiente lab](../Lab-XX-nombre/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../README.md)*  
*Licencia: [CC BY-NC-SA 4.0](../../../../LICENSE)*
