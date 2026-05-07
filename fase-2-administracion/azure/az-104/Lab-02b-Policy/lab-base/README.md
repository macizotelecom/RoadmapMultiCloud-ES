# Azure Policy — Gobernanza y Cumplimiento — Lab 02b

> Fase: F2 | Cert: AZ-104 | Lab: 02b | Tipo: base

---
**Fase:** F2 — Administración  
**Certificación:** AZ-104  
**Lab número:** 02b  
**Tipo:** base  
**Última actualización:** 2026-05  
**Costo real documentado:** [VERIFICAR COSTO]  
**Región usada:** East US  
**Duración estimada:** 35 minutos  

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Asignar una Policy built-in a nivel de Resource Group](#tarea-1)
- [Tarea 2 — Verificar compliance y entender el efecto de la Policy](#tarea-2)
- [Tarea 3 — Crear una Policy personalizada con DeployIfNotExists](#tarea-3)
- [Tarea 4 — Crear y asignar una Initiative (conjunto de políticas)](#tarea-4)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Asignar una Azure Policy built-in a un scope específico (Resource Group o Suscripción)
- [ ] Interpretar el estado de compliance de una Policy y los recursos no conformes
- [ ] Crear una Policy personalizada usando los efectos **Deny**, **Audit** y **DeployIfNotExists**
- [ ] Agrupar múltiples políticas en una **Initiative** (Policy Set) y asignarla como unidad

**Habilidades del examen que cubre este lab:**
> **Manage Azure identities and governance (20–25%)**  
> — Manage subscriptions and governance  
> — Configure Azure policies  
> — Implement and manage Azure Policy  
> — Manage policy compliance  

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 01 — Entra ID Identidades
- [ ] Lab 02a — RBAC *(familiaridad con el modelo de scope: RG → Suscripción → Management Group)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Contributor** o superior
- Para crear/asignar Policies: rol **Resource Policy Contributor** o **Owner**
- Azure CLI instalada y con sesión activa (`az login`)

**Conocimientos previos:**
- Diferencia entre RBAC (quién puede hacer qué) y Azure Policy (qué configuraciones están permitidas)
- RBAC controla acceso a operaciones; Policy controla el estado de los recursos independientemente de quién los crea

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| Azure Policy | Motor de gobernanza — evaluación y enforcement | $0.00 |
| Resource Group | Scope de asignación de políticas | $0.00 |
| Storage Account (prueba) | Recurso de test para verificar compliance | [VERIFICAR COSTO] (~$0.01 para pruebas cortas) |

> Azure Policy no tiene costo directo. El único gasto potencial es el Storage Account o recurso que crees para probar los efectos de las políticas.

---

## 🏗️ Arquitectura del lab

```
Suscripción Azure
│
├── Policy Assignment 1 (scope: Suscripción)
│   └── Initiative: "Lab02b-Baseline-Governance"
│       ├── Policy: Allowed locations → [eastus, westeurope]
│       └── Policy: Require tag Environment on resources
│
└── Resource Group: rg-lab-02b-policy
    │
    ├── Policy Assignment 2 (scope: RG — más restrictivo que suscripción)
    │   └── Policy built-in: "Audit storage accounts without https"
    │
    └── [Recursos de prueba para verificar compliance]
        └── Storage Account: stlab02b<sufijo> (creado para probar Policy)
```

**Lo que construimos en este lab:**  
Asignamos políticas a diferentes scopes para entender la jerarquía de herencia. Trabajamos con los tres efectos principales (Audit, Deny, DeployIfNotExists) y agrupamos políticas en una Initiative que representa una línea base de gobernanza. Al final tienes un modelo funcional de cómo se estructura la gobernanza en un entorno real.

**Cómo conecta con labs anteriores:**  
> El scope de Resource Group introducido en Lab 02a se reutiliza aquí: RBAC controla quién accede, Policy controla qué pueden crear. Son complementarios — en producción siempre se usan juntos.

---

## Tarea 1 — Asignar una Policy built-in a nivel de Resource Group

**Objetivo de la tarea:** Asignar la política **"Storage accounts should use customer-managed key for encryption"** en modo **Audit** sobre el RG de lab. El efecto Audit no bloquea nada — solo marca los recursos como no conformes, lo que lo hace ideal para empezar a medir el estado real del entorno sin impacto operativo.

### Método A — Portal web

1. Navega a **portal.azure.com** → busca **Policy** en la barra superior

   ![Captura paso 1](capturas/tarea1-paso-01-buscar-policy.png)

2. En el menú lateral de Policy → **Assignments** → **+ Assign policy**

   ![Captura paso 2](capturas/tarea1-paso-02-assign-policy.png)

3. En el campo **Scope** → haz clic en `...`:
   - Selecciona tu suscripción
   - Selecciona el Resource Group `rg-lab-02b-policy` (créalo si no existe)
   - Haz clic en **Select**

   ![Captura paso 3](capturas/tarea1-paso-03-scope.png)

4. En **Policy definition** → haz clic en `...` → busca `"Allowed locations"` → selecciónala

   ![Captura paso 4](capturas/tarea1-paso-04-policy-definition.png)

5. En la pestaña **Parameters**:
   - **Allowed locations:** selecciona `East US` y `West Europe`

   ![Captura paso 5](capturas/tarea1-paso-05-parametros.png)

6. En la pestaña **Remediation**: deja los valores por defecto (esta Policy es solo Audit/Deny, no necesita remediation task)

7. Haz clic en **Review + create** → **Create**

   ![Captura paso 7](capturas/tarea1-paso-07-confirmar.png)

### Método B — CLI

```powershell
# Crear el Resource Group del lab si no existe
az group create --name "rg-lab-02b-policy" --location "eastus"

$rgScope = az group show --name "rg-lab-02b-policy" --query id --output tsv

# Obtener el ID de la policy definition built-in "Allowed locations"
# La policy tiene un nombre GUID fijo en Azure
$policyId = az policy definition list `
  --query "[?displayName=='Allowed locations'].id" `
  --output tsv

# Asignar la policy al scope del RG con parámetros
# listOfAllowedLocations es el nombre del parámetro de esta policy específica
az policy assignment create `
  --name "lab02b-allowed-locations" `
  --display-name "Lab02b - Ubicaciones permitidas" `
  --policy $policyId `
  --scope $rgScope `
  --params '{"listOfAllowedLocations": {"value": ["eastus", "westeurope"]}}'

# Verificar la asignación
az policy assignment show --name "lab02b-allowed-locations" --scope $rgScope
```

### Método C — IaC (Bicep)

```bicep
// policy-assignment.bicep
// Scope: resourceGroup

// La policy "Allowed locations" tiene este ID fijo en todas las suscripciones
var allowedLocationsPolicyId = '/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4f'

resource policyAssignment 'Microsoft.Authorization/policyAssignments@2024-04-01' = {
  name: 'lab02b-allowed-locations'
  properties: {
    displayName: 'Lab02b - Ubicaciones permitidas'
    policyDefinitionId: allowedLocationsPolicyId
    parameters: {
      listOfAllowedLocations: {
        value: [
          'eastus'
          'westeurope'
        ]
      }
    }
    // Audit = registra incumplimientos sin bloquear
    // Deny = bloquea la operación si incumple
    // Para cambiar el efecto, algunos policies aceptan el parámetro 'effect'
  }
}
```

**Resultado esperado:**  
> La Policy `lab02b-allowed-locations` aparece en **Policy** → **Assignments** con scope en el Resource Group. El estado de compliance puede tardar hasta **30 minutos** en calcularse por primera vez.

---

## Tarea 2 — Verificar compliance y entender el efecto de la Policy

**Objetivo de la tarea:** Probar el comportamiento de la Policy intentando crear un recurso en una región no permitida (debe bloquearse con Deny) y revisando el dashboard de compliance. También se prueba el efecto **Audit** que registra sin bloquear.

### Método A — Portal web

1. Navega a **Policy** → **Compliance** → selecciona la asignación `lab02b-allowed-locations`

   ![Captura paso 1](capturas/tarea2-paso-01-compliance-dashboard.png)

2. Observa el estado: **Compliant** (verde) si todos los recursos del RG están en las regiones permitidas, o **Non-compliant** (rojo) si hay recursos fuera de scope

   ![Captura paso 2](capturas/tarea2-paso-02-estado-compliance.png)

3. Intenta crear un Storage Account en el RG con región `West US` (no permitida):
   - Navega al RG → **+ Create** → **Storage account**
   - Selecciona región: `West US`
   - Al hacer **Review + create**, deberías ver un error de Policy

   ![Captura paso 3](capturas/tarea2-paso-03-bloqueo-policy.png)

4. Crea el mismo Storage Account con región `East US` (permitida) — debe funcionar

   ![Captura paso 4](capturas/tarea2-paso-04-recurso-permitido.png)

### Método B — CLI

```powershell
# Intentar crear un recurso en región NO permitida — debe fallar con PolicyViolation
az storage account create `
  --name "stlab02btest$(Get-Random -Maximum 9999)" `
  --resource-group "rg-lab-02b-policy" `
  --location "westus" `
  --sku "Standard_LRS"
# Resultado esperado: error RequestDisallowedByPolicy

# Crear el recurso en región permitida — debe funcionar
$stName = "stlab02b$(Get-Random -Maximum 9999)"
az storage account create `
  --name $stName `
  --resource-group "rg-lab-02b-policy" `
  --location "eastus" `
  --sku "Standard_LRS"

# Consultar el estado de compliance de la asignación
# La evaluación puede tardar hasta 30 min — este comando fuerza una evaluación on-demand
az policy state trigger-scan --resource-group "rg-lab-02b-policy"

# Ver los recursos no conformes (si los hay)
az policy state list `
  --resource-group "rg-lab-02b-policy" `
  --filter "complianceState eq 'NonCompliant'" `
  --output table
```

**Resultado esperado:**  
> El intento de crear el Storage Account en `West US` falla con `RequestDisallowedByPolicy`. El creado en `East US` completa exitosamente. El dashboard de compliance muestra el RG como **Compliant**.

---

## Tarea 3 — Crear una Policy personalizada con DeployIfNotExists

**Objetivo de la tarea:** Crear una **Custom Policy** que, cuando se crea un Resource Group sin el tag `Environment`, lo añade automáticamente con valor `Untagged`. El efecto **DeployIfNotExists** es el más potente: no solo audita ni bloquea, sino que remedia automáticamente la desviación.

### Método A — Portal web

1. Navega a **Policy** → **Definitions** → **+ Policy definition**

   ![Captura paso 1](capturas/tarea3-paso-01-nueva-definition.png)

2. Rellena:
   - **Definition location:** tu suscripción
   - **Name:** `lab02b-require-environment-tag`
   - **Description:** `Requiere el tag Environment en Resource Groups. Si no existe, lo añade con valor Untagged.`
   - **Category:** crea nueva `Lab02b`

3. En el campo **Policy rule**, pega el JSON de la regla (ver Método B para el JSON completo)

   ![Captura paso 3](capturas/tarea3-paso-03-policy-rule.png)

4. Haz clic en **Save**

   ![Captura paso 4](capturas/tarea3-paso-04-guardar.png)

5. Asigna la policy al scope de la suscripción con una **Remediation task** habilitada:
   - **Policy** → **Assignments** → **+ Assign policy** → selecciona `lab02b-require-environment-tag`
   - En la pestaña **Remediation**: activa **Create a remediation task**
   - Asegúrate de que el Managed Identity tiene permisos para modificar Resource Groups

   ![Captura paso 5](capturas/tarea3-paso-05-remediation-task.png)

### Método B — CLI

```powershell
# Definición JSON de la policy personalizada
# DeployIfNotExists requiere un 'roleDefinitionIds' para el Managed Identity
# que ejecutará el deployment de remediación
$policyRule = @{
  mode = "All"
  policyRule = @{
    if = @{
      allOf = @(
        @{
          field = "type"
          equals = "Microsoft.Resources/subscriptions/resourceGroups"
        },
        @{
          field = "tags['Environment']"
          exists = "false"
        }
      )
    }
    then = @{
      effect = "DeployIfNotExists"
      details = @{
        type = "Microsoft.Resources/subscriptions/resourceGroups"
        # roleDefinitionIds: el Managed Identity de la policy necesita este rol
        roleDefinitionIds = @(
          "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"  # Contributor
        )
        deployment = @{
          properties = @{
            mode = "incremental"
            template = @{
              "`$schema" = "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#"
              contentVersion = "1.0.0.0"
              parameters = @{
                rgName = @{ type = "string" }
                rgLocation = @{ type = "string" }
              }
              resources = @(
                @{
                  type = "Microsoft.Resources/resourceGroups"
                  apiVersion = "2021-04-01"
                  name = "[parameters('rgName')]"
                  location = "[parameters('rgLocation')]"
                  tags = @{
                    Environment = "Untagged"
                  }
                }
              )
            }
            parameters = @{
              rgName = @{ value = "[field('name')]" }
              rgLocation = @{ value = "[field('location')]" }
            }
          }
        }
      }
    }
  }
} | ConvertTo-Json -Depth 20

$policyRule | Out-File -FilePath "custom-policy-tag.json" -Encoding utf8

# Crear la policy definition personalizada
$subscriptionId = az account show --query id --output tsv

az policy definition create `
  --name "lab02b-require-environment-tag" `
  --display-name "Lab02b - Requiere tag Environment en Resource Groups" `
  --description "Si un RG no tiene el tag Environment, lo añade con valor Untagged." `
  --rules "custom-policy-tag.json" `
  --mode "All" `
  --subscription $subscriptionId

# Crear la asignación con remediation task
# La policy con DeployIfNotExists SIEMPRE necesita un Managed Identity
az policy assignment create `
  --name "lab02b-tag-environment" `
  --display-name "Lab02b - Tag Environment en RGs" `
  --policy "lab02b-require-environment-tag" `
  --scope "/subscriptions/$subscriptionId" `
  --mi-system-assigned `
  --location "eastus"

# Obtener el principal ID del Managed Identity creado por la asignación
$miPrincipalId = az policy assignment show `
  --name "lab02b-tag-environment" `
  --scope "/subscriptions/$subscriptionId" `
  --query identity.principalId --output tsv

# Dar al Managed Identity el rol Contributor en la suscripción
# (necesario para poder modificar los Resource Groups)
az role assignment create `
  --assignee $miPrincipalId `
  --role "Contributor" `
  --scope "/subscriptions/$subscriptionId"

# Crear una remediation task para remediar recursos existentes no conformes
az policy remediation create `
  --name "lab02b-remediation-tags" `
  --policy-assignment "lab02b-tag-environment" `
  --resource-discovery-mode ExistingNonCompliant
```

**Resultado esperado:**  
> La Policy `lab02b-require-environment-tag` aparece en **Definitions** con tipo **Custom**. Al crear un nuevo Resource Group sin el tag `Environment`, la remediation task lo añade automáticamente en los siguientes minutos.

---

## Tarea 4 — Crear y asignar una Initiative (conjunto de políticas)

**Objetivo de la tarea:** Las Initiatives (Policy Sets) agrupan políticas relacionadas en una unidad que se asigna y gestiona conjuntamente. En entornos reales, siempre se trabaja con Initiatives en lugar de políticas individuales — permiten medir el compliance de un objetivo de negocio completo (ej: "cumplimiento CIS Benchmark") con una sola asignación.

### Método A — Portal web

1. Navega a **Policy** → **Definitions** → **+ Initiative definition**

   ![Captura paso 1](capturas/tarea4-paso-01-nueva-initiative.png)

2. Rellena:
   - **Name:** `lab02b-baseline-governance`
   - **Display name:** `Lab02b - Línea base de gobernanza`
   - **Category:** `Lab02b`

3. En **Policies**, añade las siguientes policies:
   - `Allowed locations` (built-in)
   - `lab02b-require-environment-tag` (la que creamos en la Tarea 3)
   - `Audit VMs that do not use managed disks` (built-in, en modo Audit)

   ![Captura paso 3](capturas/tarea4-paso-03-policies-initiative.png)

4. En **Initiative parameters**, expón `listOfAllowedLocations` como parámetro de la Initiative  
   (para poder configurarlo al asignar sin editar la Initiative)

   ![Captura paso 4](capturas/tarea4-paso-04-parameters.png)

5. Haz clic en **Save** → asigna la Initiative a la suscripción desde **Assignments**

   ![Captura paso 5](capturas/tarea4-paso-05-asignar-initiative.png)

### Método B — CLI

```powershell
# Obtener IDs de las policies built-in que incluiremos en la Initiative
$allowedLocPolicyId = az policy definition list `
  --query "[?displayName=='Allowed locations'].id" --output tsv

$managedDiskPolicyId = az policy definition list `
  --query "[?displayName=='Audit VMs that do not use managed disks'].id" --output tsv

$customTagPolicyId = az policy definition show `
  --name "lab02b-require-environment-tag" `
  --query id --output tsv

# Crear la definición JSON de la Initiative
$initiativeDef = @{
  policyDefinitions = @(
    @{
      policyDefinitionId = $allowedLocPolicyId
      parameters = @{
        listOfAllowedLocations = @{ value = "[parameters('allowedLocations')]" }
      }
    },
    @{
      policyDefinitionId = $managedDiskPolicyId
      parameters = @{}
    },
    @{
      policyDefinitionId = $customTagPolicyId
      parameters = @{}
    }
  )
  parameters = @{
    allowedLocations = @{
      type = "Array"
      metadata = @{
        displayName = "Ubicaciones permitidas"
        description = "Lista de regiones Azure donde se pueden desplegar recursos"
      }
      defaultValue = @("eastus", "westeurope")
    }
  }
} | ConvertTo-Json -Depth 10

$initiativeDef | Out-File -FilePath "initiative-baseline.json" -Encoding utf8

# Crear la Initiative definition
az policy set-definition create `
  --name "lab02b-baseline-governance" `
  --display-name "Lab02b - Linea base de gobernanza" `
  --description "Conjunto de politicas de gobernanza basica para el lab AZ-104" `
  --definitions "initiative-baseline.json" `
  --params '{"allowedLocations": {"type": "Array", "defaultValue": ["eastus", "westeurope"]}}'

# Asignar la Initiative a la suscripción
az policy assignment create `
  --name "lab02b-initiative-baseline" `
  --display-name "Lab02b - Baseline Governance Initiative" `
  --policy-set-definition "lab02b-baseline-governance" `
  --scope "/subscriptions/$subscriptionId" `
  --params '{"allowedLocations": {"value": ["eastus", "westeurope"]}}' `
  --mi-system-assigned `
  --location "eastus"

# Verificar la asignación
az policy assignment show `
  --name "lab02b-initiative-baseline" `
  --scope "/subscriptions/$subscriptionId"
```

**Resultado esperado:**  
> La Initiative `lab02b-baseline-governance` aparece en **Policy** → **Definitions** con tipo **Initiative** y 3 policies incluidas. La asignación a la suscripción es visible en **Assignments** y el dashboard de compliance empezará a mostrar resultados en ~30 minutos.

---

## ⚠️ Errores comunes

### Error 1 — `RequestDisallowedByPolicy` al crear recursos

**Síntoma:**
```
Resource 'nombre-recurso' was disallowed by policy. Policy identifiers:
[{"policyAssignment":{"name":"lab02b-allowed-locations",...}}]
```

**Causa:** Estás intentando crear un recurso en una región o con una configuración que viola alguna Policy asignada. Este es el comportamiento **correcto** cuando el efecto es `Deny`.

**Solución:**
```powershell
# Ver qué policies están afectando a tu operación
az policy state list `
  --resource-group "rg-lab-02b-policy" `
  --filter "complianceState eq 'NonCompliant'" `
  --output table
```

---

### Error 2 — La remediation task no aplica cambios

**Síntoma:** La Remediation task aparece como completada pero los recursos siguen sin el tag.

**Causa:** El Managed Identity de la Policy assignment no tiene permisos para modificar los recursos. Ocurre cuando se olvida asignar el rol al MI después de crear la asignación.

**Solución:**
```powershell
# Verificar el Managed Identity de la asignación
$miPrincipalId = az policy assignment show `
  --name "lab02b-tag-environment" `
  --scope "/subscriptions/$subscriptionId" `
  --query identity.principalId --output tsv

# Verificar si tiene rol asignado
az role assignment list --assignee $miPrincipalId --output table

# Si no tiene rol, asignarlo
az role assignment create `
  --assignee $miPrincipalId `
  --role "Contributor" `
  --scope "/subscriptions/$subscriptionId"
```

---

### Error 3 — Compliance state no se actualiza

**Síntoma:** Después de crear o modificar recursos, el dashboard de compliance sigue mostrando datos antiguos.

**Causa:** La evaluación de compliance en Azure Policy no es en tiempo real — el ciclo estándar es cada **24 horas**. Para entornos de lab, se puede forzar manualmente.

**Solución:**
```powershell
# Forzar una evaluación on-demand del RG (puede tardar 5-15 minutos)
az policy state trigger-scan `
  --resource-group "rg-lab-02b-policy"

# Para suscripción completa (más lento)
az policy state trigger-scan --subscription $subscriptionId
```

---

## 🧹 Limpieza de recursos

> ⚠️ **Importante:** Las Policy assignments permanecen activas aunque elimines los recursos. Elimina primero las asignaciones, luego las definiciones, luego los recursos.

### Método A — Portal

1. **Policy** → **Assignments** → elimina todas las asignaciones del lab (`lab02b-*`)
2. **Policy** → **Definitions** → elimina las definitions custom (`lab02b-*`)
3. Elimina el Resource Group: `rg-lab-02b-policy` → **Delete resource group**

### Método B — CLI

```powershell
# 1. Eliminar las asignaciones de policy (primero Initiative, luego policies individuales)
az policy assignment delete `
  --name "lab02b-initiative-baseline" `
  --scope "/subscriptions/$subscriptionId"

az policy assignment delete `
  --name "lab02b-tag-environment" `
  --scope "/subscriptions/$subscriptionId"

az policy assignment delete `
  --name "lab02b-allowed-locations" `
  --scope $rgScope

# 2. Eliminar las definiciones custom
az policy set-definition delete --name "lab02b-baseline-governance"
az policy definition delete --name "lab02b-require-environment-tag"

# 3. Eliminar el Resource Group y sus recursos
az group delete --name "rg-lab-02b-policy" --yes --no-wait
```

### Método C — Script automatizado

```powershell
# cleanup-lab-02b.ps1
# Limpieza completa del Lab 02b — Azure Policy

Write-Host "Iniciando limpieza del Lab 02b — Azure Policy..." -ForegroundColor Yellow

$subscriptionId = az account show --query id --output tsv
$subScope = "/subscriptions/$subscriptionId"
$rgScope = az group show --name "rg-lab-02b-policy" --query id --output tsv 2>$null

# Eliminar asignaciones de policy
$assignments = @(
    @{ name = "lab02b-initiative-baseline"; scope = $subScope },
    @{ name = "lab02b-tag-environment";     scope = $subScope },
    @{ name = "lab02b-allowed-locations";   scope = $rgScope }
)

foreach ($a in $assignments) {
    if ($a.scope) {
        az policy assignment delete --name $a.name --scope $a.scope 2>$null
        Write-Host "Assignment '$($a.name)' eliminado." -ForegroundColor Green
    }
}

# Eliminar definiciones custom (Initiative primero, luego Policy)
az policy set-definition delete --name "lab02b-baseline-governance" 2>$null
Write-Host "Initiative definition eliminada." -ForegroundColor Green

az policy definition delete --name "lab02b-require-environment-tag" 2>$null
Write-Host "Custom policy definition eliminada." -ForegroundColor Green

# Eliminar Resource Group
if ($rgScope) {
    az group delete --name "rg-lab-02b-policy" --yes --no-wait
    Write-Host "Resource Group enviado a eliminacion (async)." -ForegroundColor Green
}

Write-Host "Limpieza completada." -ForegroundColor Yellow
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| Azure Policy (asignaciones y evaluaciones) | — | $0.00 |
| Storage Account de prueba (Standard LRS, East US) | ~30 min | [VERIFICAR COSTO] |
| Resource Group | ~35 min | $0.00 |
| **Total lab** | | **~$0.00–$0.01** |

**Captura del billing:**

![Billing del lab](capturas/billing-lab-02b.png)

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]  
**Región:** East US  
**Notas de costo:**
> Azure Policy no genera costo. El Storage Account creado para probar los efectos es mínimo (~$0.01 o menos en 30 minutos). Si se limpia inmediatamente, el costo puede ser $0.00 por redondeo de billing.

---

## 📊 Análisis FinOps rápido

*Para el análisis FinOps completo ver: [finops/README.md](../../finops/README.md)*

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Servicio de gobernanza | Azure Policy | AWS Config + SCPs | Organization Policy | OCI Security Zones |
| Políticas predefinidas | ~500 built-in | ~300 managed rules | ~100 constraints | ~50 policies |
| Efecto Deny | Sí (nativo) | Sí via SCPs | Sí via constraints | Sí via Security Zones |
| Remediación automática | Sí (DeployIfNotExists) | Sí via Config Rules | Parcial (Recommender) | Limitada |
| Costo del servicio | $0.00 | AWS Config: ~$0.003/evaluation | $0.00 | $0.00 |

**¿Cuándo elegir Azure Policy vs alternativas?**  
> Azure Policy es la opción más madura para entornos híbridos con on-premises, especialmente si ya se usa Azure Arc para extender la gobernanza a recursos fuera de Azure. AWS Config es comparable pero requiere combinar con SCPs (Service Control Policies) para obtener el efecto Deny equivalente. GCP Organization Policy es más limitada en efectos de remediación automática.

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- La diferencia entre **Audit** (observa sin bloquear), **Deny** (bloquea la operación) y **DeployIfNotExists** (remedia automáticamente) — elegir el efecto correcto es crítico: empezar con Audit antes de pasar a Deny
- Cómo las Policies se heredan por jerarquía de scope y cómo una Policy más restrictiva en el RG sobreescribe una más permisiva de la suscripción
- Por qué las **Initiatives** son la unidad de trabajo real en producción: agrupan políticas relacionadas y permiten medir compliance de un objetivo de negocio completo
- El ciclo de evaluación de compliance (~24h) y cómo forzarlo on-demand con `az policy state trigger-scan`

**Cómo se usa en labs futuros:**
> Las políticas de ubicación y tagging establecidas aquí se asumen activas en **Lab 07 — Storage** y **Lab 08 — VMs**: todos los recursos de labs posteriores deben crearse en `East US` o `West Europe` y con el tag `Environment` correctamente configurado.

**Siguiente lab recomendado:**  
👉 [Lab 03 — ARM Templates y Bicep](../../Lab-03-ARM-Bicep/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*  
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
