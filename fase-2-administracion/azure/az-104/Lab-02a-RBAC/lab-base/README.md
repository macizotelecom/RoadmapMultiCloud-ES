# RBAC — Control de Acceso Basado en Roles — Lab 02a

> Fase: F2 | Cert: AZ-104 | Lab: 02a | Tipo: base

---
**Fase:** F2 — Administración  
**Certificación:** AZ-104  
**Lab número:** 02a  
**Tipo:** base  
**Última actualización:** 2026-05  
**Costo real documentado:** [VERIFICAR COSTO]  
**Región usada:** East US  
**Duración estimada:** 30 minutos  

---

## 📋 Índice

- [Objetivos](#objetivos)
- [Prerequisitos](#prerequisitos)
- [Servicios utilizados](#servicios-utilizados)
- [Arquitectura del lab](#arquitectura-del-lab)
- [Tarea 1 — Crear un Resource Group de ámbito para RBAC](#tarea-1)
- [Tarea 2 — Asignar un rol RBAC a nivel de Resource Group](#tarea-2)
- [Tarea 3 — Verificar la asignación de rol y comprobar permisos efectivos](#tarea-3)
- [Tarea 4 — Crear un rol personalizado con permisos acotados](#tarea-4)
- [Errores comunes](#errores-comunes)
- [Limpieza de recursos](#limpieza-de-recursos)
- [Costo real](#costo-real-documentado)
- [Análisis FinOps rápido](#análisis-finops-rápido)
- [Resumen y siguiente lab](#resumen-y-siguiente-lab)

---

## 🎯 Objetivos

Al completar este lab serás capaz de:

- [ ] Asignar roles RBAC integrados (built-in) a usuarios y grupos en diferentes ámbitos (subscription, resource group, resource)
- [ ] Verificar los permisos efectivos de un principal de seguridad usando el portal y la CLI
- [ ] Crear un rol personalizado (custom role) con un conjunto acotado de permisos
- [ ] Entender la diferencia entre ámbito (scope), rol y asignación de rol

**Habilidades del examen que cubre este lab:**
> **Manage Azure identities and governance (20–25%)**  
> — Manage role-based access control (RBAC)  
> — Assign Azure roles  
> — Create and assign custom roles  
> — Interpret access assignments  

---

## ✅ Prerequisitos

**Labs anteriores requeridos:**
- [ ] Lab 01 — Entra ID Identidades *(necesitas usuarios y grupos creados en ese lab)*

**Acceso necesario:**
- Suscripción Azure activa con rol **Owner** o **User Access Administrator** (necesario para asignar roles)
- Azure CLI instalada y con sesión activa (`az login`)

**Conocimientos previos:**
- Concepto de principal de seguridad en Azure (usuario, grupo, service principal, managed identity)
- Diferencia entre autenticación (quién eres) y autorización (qué puedes hacer)
- Modelo de herencia de permisos en Azure: Management Group → Subscription → Resource Group → Resource

---

## 🛠️ Servicios utilizados

| Servicio | Propósito en este lab | Costo aprox. |
|----------|----------------------|--------------|
| Azure RBAC | Motor de autorización — no genera costo directo | $0.00 |
| Microsoft Entra ID | Resolución de identidades (usuarios/grupos) | $0.00 (tier free) |
| Resource Group | Ámbito de las asignaciones de rol | $0.00 |

> RBAC en sí mismo no tiene costo. El costo potencial viene de los recursos que se creen dentro del scope, que en este lab no incluyen compute ni storage.

---

## 🏗️ Arquitectura del lab

```
Suscripción Azure
│
└── Resource Group: rg-lab-02a-rbac
    │
    ├── [Scope de asignación de roles]
    │   ├── Rol: Reader          → Usuario: az104-user01 (del Lab 01)
    │   ├── Rol: Contributor     → Grupo:   az104-admins (del Lab 01)
    │   └── Rol: Custom-VMOps    → Usuario: az104-user02 (del Lab 01)
    │
    └── [Sin recursos de compute — solo el RG como ámbito]
```

**Lo que construimos en este lab:**  
Un Resource Group limpio usado exclusivamente como ámbito de asignaciones RBAC. Asignamos roles built-in a los usuarios y grupos creados en el Lab 01, verificamos los permisos efectivos resultantes, y creamos un rol personalizado que permite operaciones específicas sobre VMs sin dar acceso completo de Contributor.

**Cómo conecta con labs anteriores:**  
> Usa los usuarios (`az104-user01`, `az104-user02`) y el grupo (`az104-admins`) creados en el **Lab 01 — Entra ID**. Si no tienes esos objetos, créalos antes de continuar.

---

## Tarea 1 — Crear un Resource Group de ámbito para RBAC

**Objetivo de la tarea:** Crear el Resource Group que actuará como scope de todas las asignaciones de rol de este lab. Trabajar a nivel de RG (en lugar de suscripción) es la práctica correcta: principio de mínimo privilegio + aislamiento de blast radius.

### Método A — Portal web

1. Navega a **portal.azure.com** → busca **Resource groups** en la barra superior

   ![Captura paso 1](capturas/tarea1-paso-01-buscar-rg.png)

2. Haz clic en **+ Create**

   ![Captura paso 2](capturas/tarea1-paso-02-crear-rg.png)

3. Rellena los campos:
   - **Subscription:** tu suscripción activa
   - **Resource group:** `rg-lab-02a-rbac`
   - **Region:** `East US`

   ![Captura paso 3](capturas/tarea1-paso-03-campos-rg.png)

4. Haz clic en **Review + create** → **Create**

   ![Captura paso 4](capturas/tarea1-paso-04-resultado-rg.png)

### Método B — CLI

```powershell
# Crear el resource group de ámbito para este lab
# --location define la región de metadatos del RG (no de los recursos dentro)
az group create `
  --name "rg-lab-02a-rbac" `
  --location "eastus"

# Verificar que se creó correctamente
az group show --name "rg-lab-02a-rbac" --output table
```

### Método C — IaC (Bicep)

```bicep
// main.bicep
// Scope: subscription (targetScope obligatorio para crear RGs)
targetScope = 'subscription'

param location string = 'eastus'
param rgName string = 'rg-lab-02a-rbac'

resource resourceGroup 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: rgName
  location: location
}
```

```powershell
# Desplegar el Bicep a nivel de suscripción
az deployment sub create `
  --location "eastus" `
  --template-file "main.bicep"
```

**Resultado esperado:**  
> El Resource Group `rg-lab-02a-rbac` aparece en la lista de Resource Groups con estado **Succeeded** y región **East US**.

---

## Tarea 2 — Asignar un rol RBAC a nivel de Resource Group

**Objetivo de la tarea:** Asignar el rol **Reader** a `az104-user01` y el rol **Contributor** al grupo `az104-admins` sobre el scope del Resource Group. Esto demuestra las dos unidades de asignación más comunes: usuario individual vs grupo (preferible en entornos reales porque escala mejor).

### Método A — Portal web

1. Navega al Resource Group `rg-lab-02a-rbac` → selecciona **Access control (IAM)** en el menú lateral

   ![Captura paso 1](capturas/tarea2-paso-01-iam-menu.png)

2. Haz clic en **+ Add** → **Add role assignment**

   ![Captura paso 2](capturas/tarea2-paso-02-add-role.png)

3. En la pestaña **Role**, busca y selecciona **Reader** → **Next**

   ![Captura paso 3](capturas/tarea2-paso-03-seleccionar-reader.png)

4. En la pestaña **Members**:
   - **Assign access to:** `User, group, or service principal`
   - Haz clic en **+ Select members** → busca `az104-user01` → selecciona → **Select**

   ![Captura paso 4](capturas/tarea2-paso-04-seleccionar-user01.png)

5. Haz clic en **Review + assign** → **Review + assign** (confirma dos veces)

   ![Captura paso 5](capturas/tarea2-paso-05-confirmar-reader.png)

6. Repite los pasos 2–5 para asignar el rol **Contributor** al grupo `az104-admins`

   ![Captura paso 6](capturas/tarea2-paso-06-contributor-admins.png)

7. En la pestaña **Role assignments** verifica que aparecen ambas asignaciones

   ![Captura paso 7](capturas/tarea2-paso-07-verificar-asignaciones.png)

### Método B — CLI

```powershell
# Obtener el ID del Resource Group (scope para la asignación)
$scope = az group show --name "rg-lab-02a-rbac" --query id --output tsv

# Obtener el Object ID de az104-user01
$user01Id = az ad user show --id "az104-user01@<tudominio>.onmicrosoft.com" `
  --query id --output tsv

# Asignar rol Reader a az104-user01 en el scope del RG
# --role acepta nombre del rol o su GUID
az role assignment create `
  --assignee $user01Id `
  --role "Reader" `
  --scope $scope

# Obtener el Object ID del grupo az104-admins
$groupId = az ad group show --group "az104-admins" --query id --output tsv

# Asignar rol Contributor al grupo az104-admins en el scope del RG
az role assignment create `
  --assignee $groupId `
  --role "Contributor" `
  --scope $scope

# Verificar las asignaciones creadas
az role assignment list --scope $scope --output table
```

### Método C — IaC (Bicep)

```bicep
// rbac-assignments.bicep
// Scope: resourceGroup (default en Bicep)

// El GUID del rol Reader es fijo en Azure — no cambia entre tenants
var readerRoleId = 'acdd72a7-3385-48ef-bd42-f606fba81ae7'
// El GUID del rol Contributor
var contributorRoleId = 'b24988ac-6180-42a0-ab88-20f7382dd24c'

param user01ObjectId string  // Object ID de az104-user01
param adminsGroupObjectId string  // Object ID del grupo az104-admins

// Asignación Reader → az104-user01
// El name del resource DEBE ser un GUID — se usa guid() para generarlo deterministicamente
resource readerAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(resourceGroup().id, user01ObjectId, readerRoleId)
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', readerRoleId)
    principalId: user01ObjectId
    principalType: 'User'  // Importante: evita ambigüedad si hay SP con mismo ID
  }
}

// Asignación Contributor → grupo az104-admins
resource contributorAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(resourceGroup().id, adminsGroupObjectId, contributorRoleId)
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', contributorRoleId)
    principalId: adminsGroupObjectId
    principalType: 'Group'
  }
}
```

**Resultado esperado:**  
> En la pestaña **Role assignments** del IAM del RG aparecen dos entradas: `az104-user01` con rol **Reader** y `az104-admins` con rol **Contributor**. El scope de ambas es el Resource Group (no la suscripción).

---

## Tarea 3 — Verificar la asignación de rol y comprobar permisos efectivos

**Objetivo de la tarea:** Confirmar que RBAC funciona como se diseñó. Hay dos vectores de verificación: (1) ver las asignaciones desde el owner, y (2) comprobar qué puede hacer el usuario asignado. El segundo es el que realmente valida el comportamiento en el examen.

### Método A — Portal web

1. En el Resource Group `rg-lab-02a-rbac` → **Access control (IAM)** → pestaña **Check access**

   ![Captura paso 1](capturas/tarea3-paso-01-check-access.png)

2. En el buscador escribe `az104-user01` → selecciona el usuario

   ![Captura paso 2](capturas/tarea3-paso-02-buscar-usuario.png)

3. Verifica que aparece el rol **Reader** con scope en el Resource Group

   ![Captura paso 3](capturas/tarea3-paso-03-permisos-efectivos.png)

4. Abre una sesión de incógnito → inicia sesión como `az104-user01` → navega al RG  
   Verifica que puede **ver** los recursos pero el botón **+ Create** está deshabilitado

   ![Captura paso 4](capturas/tarea3-paso-04-vista-reader.png)

### Método B — CLI

```powershell
# Listar todas las asignaciones de rol en el scope del RG con detalle
az role assignment list `
  --scope $scope `
  --include-inherited `
  --output table

# Verificar los permisos efectivos de az104-user01 en el RG
# Requiere: az role assignment list + cruce con la definición del rol
az role definition list --name "Reader" --query "[].permissions" --output json

# Ver qué acciones permite el rol Reader (actions vs notActions)
az role definition list `
  --name "Reader" `
  --query "[0].{Actions:permissions[0].actions, NotActions:permissions[0].notActions}" `
  --output json
```

### Método C — IaC (verificación)

```powershell
# No hay IaC para verificación — se hace via CLI o portal
# Pero puedes exportar el estado actual de asignaciones para auditoría:
az role assignment list `
  --scope $scope `
  --output json | Out-File -FilePath "rbac-audit-$(Get-Date -Format 'yyyyMMdd').json"
```

**Resultado esperado:**  
> `az104-user01` puede navegar al RG y ver recursos existentes, pero no puede crear, modificar ni eliminar nada. El portal muestra los botones de acción deshabilitados o ausentes.

---

## Tarea 4 — Crear un rol personalizado con permisos acotados

**Objetivo de la tarea:** Crear un **Custom Role** llamado `VM-Operator-Lab02a` que permite iniciar, detener y reiniciar VMs, pero no crearlas ni eliminarlas. Este es el patrón más frecuente en entornos reales: el Contributor da demasiado, el Reader da demasiado poco — el custom role cubre exactamente lo necesario.

### Método A — Portal web

1. Navega a la **Suscripción** → **Access control (IAM)** → **+ Add** → **Add custom role**

   ![Captura paso 1](capturas/tarea4-paso-01-add-custom-role.png)

2. Selecciona **Start from scratch** → **Next**

   ![Captura paso 2](capturas/tarea4-paso-02-scratch.png)

3. En la pestaña **Basics**:
   - **Custom role name:** `VM-Operator-Lab02a`
   - **Description:** `Permite iniciar, detener y reiniciar VMs. Sin permisos de creación ni eliminación.`

   ![Captura paso 3](capturas/tarea4-paso-03-basics.png)

4. En la pestaña **Permissions** → **+ Add permissions** → busca `Microsoft.Compute/virtualMachines`  
   Selecciona únicamente:
   - `Microsoft.Compute/virtualMachines/start/action`
   - `Microsoft.Compute/virtualMachines/deallocate/action`
   - `Microsoft.Compute/virtualMachines/restart/action`
   - `Microsoft.Compute/virtualMachines/read`

   ![Captura paso 4](capturas/tarea4-paso-04-permissions.png)

5. En la pestaña **Assignable scopes** verifica que incluye tu suscripción → **Next**

   ![Captura paso 5](capturas/tarea4-paso-05-scopes.png)

6. En **Review + create** → **Create**

   ![Captura paso 6](capturas/tarea4-paso-06-crear-rol.png)

### Método B — CLI

```powershell
# Obtener el ID de la suscripción actual
$subscriptionId = az account show --query id --output tsv

# Crear el archivo de definición del rol en JSON
# Azure requiere este formato exacto para custom roles via CLI
$roleDef = @{
  Name = "VM-Operator-Lab02a"
  Description = "Permite iniciar, detener y reiniciar VMs. Sin permisos de creacion ni eliminacion."
  Actions = @(
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"  # necesario para navegar al RG
  )
  NotActions = @()
  AssignableScopes = @("/subscriptions/$subscriptionId")
} | ConvertTo-Json -Depth 5

$roleDef | Out-File -FilePath "vm-operator-role.json" -Encoding utf8

# Crear el rol personalizado en Azure
az role definition create --role-definition "vm-operator-role.json"

# Verificar que el rol se creó correctamente
az role definition list --name "VM-Operator-Lab02a" --output table

# Asignar el rol personalizado a az104-user02 en el scope del RG
$user02Id = az ad user show --id "az104-user02@<tudominio>.onmicrosoft.com" `
  --query id --output tsv

az role assignment create `
  --assignee $user02Id `
  --role "VM-Operator-Lab02a" `
  --scope $scope
```

### Método C — IaC (Bicep)

```bicep
// custom-role.bicep
// Los custom roles se definen a nivel de suscripción
targetScope = 'subscription'

var subscriptionId = subscription().subscriptionId

resource vmOperatorRole 'Microsoft.Authorization/roleDefinitions@2022-04-01' = {
  // El GUID debe ser único y estable — generarlo una vez y hardcodearlo
  name: guid('VM-Operator-Lab02a', subscriptionId)
  properties: {
    roleName: 'VM-Operator-Lab02a'
    description: 'Permite iniciar, detener y reiniciar VMs. Sin permisos de creacion ni eliminacion.'
    type: 'CustomRole'
    permissions: [
      {
        actions: [
          'Microsoft.Compute/virtualMachines/start/action'
          'Microsoft.Compute/virtualMachines/deallocate/action'
          'Microsoft.Compute/virtualMachines/restart/action'
          'Microsoft.Compute/virtualMachines/read'
          'Microsoft.Resources/subscriptions/resourceGroups/read'
        ]
        notActions: []
        dataActions: []
        notDataActions: []
      }
    ]
    assignableScopes: [
      '/subscriptions/${subscriptionId}'
    ]
  }
}
```

**Resultado esperado:**  
> El rol `VM-Operator-Lab02a` aparece en la lista de **Custom roles** dentro de IAM de la suscripción. Al asignarlo a `az104-user02` y verificar con **Check access**, muestra exactamente las cuatro acciones definidas — ni más ni menos.

---

## ⚠️ Errores comunes

### Error 1 — `PrincipalNotFound` al asignar rol via CLI

**Síntoma:**
```
(PrincipalNotFound) Principal <GUID> does not exist in the directory.
```

**Causa:** El Object ID que estás pasando no existe en el tenant, o hay un retraso de replicación de Entra ID. Ocurre frecuentemente cuando el usuario se acaba de crear (Lab 01) y Entra ID aún no lo ha propagado.

**Solución:**
```powershell
# Esperar 30-60 segundos y volver a obtener el Object ID fresco
Start-Sleep -Seconds 60
$user01Id = az ad user show --id "az104-user01@<tudominio>.onmicrosoft.com" `
  --query id --output tsv
```

---

### Error 2 — `AuthorizationFailed` al crear el custom role

**Síntoma:**
```
The client does not have authorization to perform action
'Microsoft.Authorization/roleDefinitions/write' over scope '/subscriptions/...'
```

**Causa:** Tu cuenta no tiene el rol **Owner** o **User Access Administrator** a nivel de suscripción. El rol **Contributor** no es suficiente para gestionar asignaciones RBAC.

**Solución:** Verifica tu rol en la suscripción:
```powershell
az role assignment list --assignee $(az account show --query user.name --output tsv) `
  --scope "/subscriptions/$(az account show --query id --output tsv)" `
  --output table
```

---

### Error 3 — El custom role no aparece disponible para asignar

**Síntoma:** Al intentar asignar `VM-Operator-Lab02a`, el rol no aparece en el buscador del portal.

**Causa:** La propagación de custom roles puede tardar hasta **5 minutos**. También puede ocurrir si el `assignableScopes` no incluye el scope donde intentas asignarlo.

**Solución:**
```powershell
# Verificar que el rol existe y su assignableScopes es correcto
az role definition list --name "VM-Operator-Lab02a" `
  --query "[0].{Name:roleName, Scopes:assignableScopes}" `
  --output json
# Si es necesario, esperar y reintentar
```

---

## 🧹 Limpieza de recursos

> ⚠️ **Importante:** RBAC no genera costo directo, pero es buena práctica limpiar las asignaciones y el custom role para no contaminar el tenant con roles de prueba.

### Método A — Portal

1. Navega al Resource Group `rg-lab-02a-rbac` → **Access control (IAM)** → **Role assignments**
2. Selecciona todas las asignaciones creadas en este lab → **Remove**
3. Navega a la **Suscripción** → **IAM** → **Roles** → busca `VM-Operator-Lab02a` → **...** → **Delete**
4. Elimina el Resource Group: `rg-lab-02a-rbac` → **Delete resource group**

### Método B — CLI

```powershell
# 1. Eliminar las asignaciones de rol en el RG
az role assignment delete `
  --assignee $user01Id `
  --role "Reader" `
  --scope $scope

az role assignment delete `
  --assignee $groupId `
  --role "Contributor" `
  --scope $scope

az role assignment delete `
  --assignee $user02Id `
  --role "VM-Operator-Lab02a" `
  --scope $scope

# 2. Eliminar el custom role
az role definition delete --name "VM-Operator-Lab02a"

# 3. Eliminar el Resource Group
az group delete --name "rg-lab-02a-rbac" --yes --no-wait
```

### Método C — Script automatizado

```powershell
# cleanup-lab-02a.ps1
# Limpieza completa del Lab 02a — RBAC

Write-Host "Iniciando limpieza del Lab 02a — RBAC..." -ForegroundColor Yellow

$scope = az group show --name "rg-lab-02a-rbac" --query id --output tsv 2>$null

if ($scope) {
    # Eliminar todas las asignaciones en el scope del RG de una vez
    $assignments = az role assignment list --scope $scope --query "[].id" --output tsv
    foreach ($id in $assignments) {
        az role assignment delete --ids $id
    }
    Write-Host "Asignaciones de rol eliminadas." -ForegroundColor Green

    # Eliminar custom role
    az role definition delete --name "VM-Operator-Lab02a" 2>$null
    Write-Host "Custom role eliminado." -ForegroundColor Green

    # Eliminar Resource Group
    az group delete --name "rg-lab-02a-rbac" --yes --no-wait
    Write-Host "Resource Group enviado a eliminacion (async)." -ForegroundColor Green
} else {
    Write-Host "Resource Group no encontrado — puede que ya este eliminado." -ForegroundColor Red
}

Write-Host "Limpieza completada." -ForegroundColor Yellow
```

---

## 💰 Costo real documentado

| Concepto | Duración | Costo |
|----------|----------|-------|
| Azure RBAC (asignaciones de rol) | — | $0.00 |
| Microsoft Entra ID (tier free) | — | $0.00 |
| Resource Group (sin recursos dentro) | ~30 min | $0.00 |
| **Total lab** | | **$0.00** |

**Captura del billing:**

![Billing del lab](capturas/billing-lab-02a.png)

**Fecha de ejecución:** [VERIFICAR COSTO — completar al ejecutar]  
**Región:** East US  
**Notas de costo:**
> Este lab no genera costo porque RBAC y Entra ID (tier free) no se facturan por asignaciones. El único costo potencial sería si creas recursos compute dentro del RG para probar los permisos, pero no forma parte de las tareas de este lab.

---

## 📊 Análisis FinOps rápido

*Para el análisis FinOps completo ver: [finops/README.md](../../finops/README.md)*

| Concepto | Azure | AWS | GCP | OCI |
|----------|-------|-----|-----|-----|
| Servicio de autorización | Azure RBAC | IAM (Policies) | Cloud IAM | OCI IAM |
| Roles predefinidos | ~100 built-in roles | ~750 managed policies | ~100 predefined roles | ~70 built-in policies |
| Roles personalizados | Sí (custom roles) | Sí (custom policies) | Sí (custom roles) | Sí (custom policies) |
| Costo del servicio | $0.00 | $0.00 | $0.00 | $0.00 |
| Granularidad | Acción → Resource → RG → Sub | Acción → ARN → Account | Acción → Proyecto → Org | Acción → Compartment |

**¿Cuándo elegir Azure RBAC vs alternativas?**  
> Azure RBAC es la opción natural si ya estás en el ecosistema Microsoft/Azure. Su integración con Entra ID y la herencia de permisos por Resource Group es más intuitiva que el modelo de IAM de AWS para entornos híbridos con Active Directory on-premises. GCP IAM y OCI IAM son comparables en funcionalidad pero con modelos de organización jerárquica distintos.

---

## 📝 Resumen y siguiente lab

**Lo que aprendiste en este lab:**
- La diferencia entre **scope** (dónde aplica), **role** (qué permite) y **role assignment** (la unión de ambos)
- Cómo asignar roles built-in a usuarios y grupos, y por qué los grupos escalan mejor
- Cómo construir un custom role con el **principio de mínimo privilegio**: solo las acciones exactas necesarias
- Cómo verificar permisos efectivos con **Check access** y con `az role assignment list`

**Cómo se usa en labs futuros:**
> El modelo RBAC establecido aquí (usuarios del Lab 01 + roles personalizados) es la base de control de acceso que se usa en **Lab 08 — VMs** para restringir quién puede operar las máquinas virtuales, y en **Lab 11 — Monitoring** para dar acceso de solo lectura al panel de métricas.

**Siguiente lab recomendado:**  
👉 [Lab 02b — Azure Policy](../../Lab-02b-Policy/lab-base/README.md)

---

*Lab documentado por el proyecto [RoadmapMultiCloud-ES](../../../../../../README.md)*  
*Licencia: [CC BY-NC-SA 4.0](../../../../../../LICENSE)*
