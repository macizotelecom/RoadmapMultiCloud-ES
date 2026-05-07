# Lab 01 Avanzado — Identidades Entra ID: escenario empresa

> Fase: F2 | Cert: AZ-104 | Lab: 01 | Tipo: avanzado

---

**Prerequisito:** [Lab-01 base](../lab-base/README.md) completado  
**Duración estimada:** 45 minutos  
**Costo real documentado:** $0.00 — sin recursos de compute  

---

## Escenario: TechNova Solutions

**TechNova Solutions** es una empresa de desarrollo de software con 80 empleados
distribuidos en tres departamentos: IT, Development y Finance.
Acaba de migrar su infraestructura on-premises a Azure y necesita estructurar
su directorio de identidades desde cero.

Este escenario ficticio se usará en todos los labs del AZ-104.
Cada lab añade una capa de infraestructura sobre lo que existe.

### Situación inicial

El administrador de sistemas (tú) recibe este requerimiento del CTO:

> "Necesitamos que el acceso a los recursos de Azure esté organizado por
> departamento desde el primer día. IT debe poder administrar la plataforma,
> Development debe tener acceso solo a sus entornos, y Finance solo a sus
> reportes. Además tenemos tres consultores externos que necesitan acceso
> limitado durante los próximos 3 meses."

### Lo que construimos en este lab
Tenant: technova.onmicrosoft.com
│
├── Usuarios internos (5)
│   ├── IT: it-admin01, it-admin02
│   ├── Development: dev-lead01, dev-user01
│   └── Finance: fin-user01
│
├── Usuarios invitados B2B (3)
│   ├── consultant01@external.com
│   ├── consultant02@external.com
│   └── consultant03@external.com
│
└── Grupos de seguridad (4)
├── TN-IT-Administrators     → miembros: it-admin01, it-admin02
├── TN-Dev-Team              → miembros: dev-lead01, dev-user01
├── TN-Finance-Team          → miembros: fin-user01
└── TN-External-Consultants  → miembros: consultant01, 02, 03

---

## Tarea 1 — Crear usuarios por departamento con PowerShell

En entornos reales no se crean usuarios uno a uno desde el portal.
Este script crea los 5 usuarios internos de TechNova en un solo bloque.

Requiere módulo Microsoft.Graph:

```powershell
# Instalar si no está disponible
Install-Module Microsoft.Graph -Scope CurrentUser

Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"

$domain = (Get-MgDomain | Where-Object { $_.IsDefault }).Id

$usuarios = @(
    @{ UPN="it-admin01"; Nombre="IT Admin 01"; Titulo="IT Administrator"; Dept="IT" },
    @{ UPN="it-admin02"; Nombre="IT Admin 02"; Titulo="IT Administrator"; Dept="IT" },
    @{ UPN="dev-lead01"; Nombre="Dev Lead 01"; Titulo="Developer Lead";   Dept="Development" },
    @{ UPN="dev-user01"; Nombre="Dev User 01"; Titulo="Developer";        Dept="Development" },
    @{ UPN="fin-user01"; Nombre="Fin User 01"; Titulo="Finance Analyst";  Dept="Finance" }
)

foreach ($u in $usuarios) {
    $passwordProfile = @{
        forceChangePasswordNextSignIn = $true
        password = "TechNova2026!"
    }
    New-MgUser `
        -DisplayName $u.Nombre `
        -UserPrincipalName "$($u.UPN)@$domain" `
        -PasswordProfile $passwordProfile `
        -AccountEnabled `
        -JobTitle $u.Titulo `
        -Department $u.Dept `
        -UsageLocation "US"
    Write-Host "Creado: $($u.UPN)@$domain"
}
```

Verificación:

```powershell
Get-MgUser -All |
    Where-Object { $_.Department -in @("IT","Development","Finance") } |
    Select-Object DisplayName, UserPrincipalName, Department, JobTitle |
    Sort-Object Department |
    Format-Table -AutoSize
```

---

## Tarea 2 — Invitar consultores externos

```powershell
$consultores = @(
    "consultant01@outlook.com",
    "consultant02@outlook.com",
    "consultant03@outlook.com"
)

foreach ($email in $consultores) {
    az ad invitation create `
        --invited-user-email-address $email `
        --invite-redirect-url "https://myapps.microsoft.com" `
        --send-invitation-message true
    Write-Host "Invitación enviada a: $email"
}
```

> **Nota de seguridad:** en producción los usuarios guest deben tener **Access Reviews**
> configurados para que el acceso expire automáticamente al finalizar el contrato.
> Esto se configura en Entra ID → Identity Governance.

---

## Tarea 3 — Crear grupos de seguridad por departamento

```powershell
$grupos = @(
    @{ Nombre="TN-IT-Administrators";    Desc="TechNova IT administrators" },
    @{ Nombre="TN-Dev-Team";             Desc="TechNova development team" },
    @{ Nombre="TN-Finance-Team";         Desc="TechNova finance team" },
    @{ Nombre="TN-External-Consultants"; Desc="TechNova external consultants B2B" }
)

$grupoIds = @{}

foreach ($g in $grupos) {
    $grupo = New-MgGroup `
        -DisplayName $g.Nombre `
        -MailNickname ($g.Nombre -replace "-","") `
        -Description $g.Desc `
        -SecurityEnabled `
        -MailEnabled:$false `
        -GroupTypes @()
    $grupoIds[$g.Nombre] = $grupo.Id
    Write-Host "Grupo creado: $($g.Nombre)"
}
```

---

## Tarea 4 — Asignar miembros a grupos

```powershell
function Get-UserId($upn) {
    return (Get-MgUser -Filter "userPrincipalName eq '$upn'").Id
}

$domain = (Get-MgDomain | Where-Object { $_.IsDefault }).Id

New-MgGroupMember -GroupId $grupoIds["TN-IT-Administrators"] -DirectoryObjectId (Get-UserId "it-admin01@$domain")
New-MgGroupMember -GroupId $grupoIds["TN-IT-Administrators"] -DirectoryObjectId (Get-UserId "it-admin02@$domain")
New-MgGroupMember -GroupId $grupoIds["TN-Dev-Team"]          -DirectoryObjectId (Get-UserId "dev-lead01@$domain")
New-MgGroupMember -GroupId $grupoIds["TN-Dev-Team"]          -DirectoryObjectId (Get-UserId "dev-user01@$domain")
New-MgGroupMember -GroupId $grupoIds["TN-Finance-Team"]      -DirectoryObjectId (Get-UserId "fin-user01@$domain")

foreach ($nombreGrupo in $grupoIds.Keys) {
    Write-Host "`n--- $nombreGrupo ---"
    Get-MgGroupMember -GroupId $grupoIds[$nombreGrupo] |
        ForEach-Object { Get-MgUser -UserId $_.Id } |
        Select-Object DisplayName, UserPrincipalName |
        Format-Table -AutoSize
}
```

---

## Tarea 5 — Verificación final

```powershell
Write-Host "=== RESUMEN TECHNOVA SOLUTIONS ===" -ForegroundColor Cyan

Write-Host "`nUsuarios internos por departamento:"
Get-MgUser -All |
    Where-Object { $_.Department -in @("IT","Development","Finance") } |
    Select-Object Department, DisplayName, JobTitle |
    Sort-Object Department |
    Format-Table -AutoSize

Write-Host "`nUsuarios guest (B2B):"
Get-MgUser -All |
    Where-Object { $_.UserType -eq "Guest" } |
    Select-Object DisplayName, UserPrincipalName |
    Format-Table -AutoSize

Write-Host "`nGrupos TechNova:"
Get-MgGroup -All |
    Where-Object { $_.DisplayName -like "TN-*" } |
    Select-Object DisplayName, Description |
    Format-Table -AutoSize
```

---

## Troubleshooting

### Connect-MgGraph no se reconoce

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Insufficient privileges al crear usuarios

El usuario no tiene rol **User Administrator** o **Global Administrator**.  
Verificar en Entra ID → Users → tu cuenta → Assigned roles.

### Authorization_RequestDenied en invitación B2B

El tenant tiene restringidas las invitaciones externas.  
Solución: Entra ID → External Identities → External collaboration settings →
verificar que Guest invite settings permite invitaciones desde administradores.

---

## Limpieza de recursos

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"
$domain = (Get-MgDomain | Where-Object { $_.IsDefault }).Id

Get-MgGroup -All | Where-Object { $_.DisplayName -like "TN-*" } | ForEach-Object {
    Remove-MgGroup -GroupId $_.Id
    Write-Host "Eliminado grupo: $($_.DisplayName)"
}

@("it-admin01","it-admin02","dev-lead01","dev-user01","fin-user01") | ForEach-Object {
    $user = Get-MgUser -Filter "userPrincipalName eq '$_@$domain'"
    if ($user) {
        Remove-MgUser -UserId $user.Id
        Write-Host "Eliminado usuario: $_"
    }
}
```

---

## Cómo se usa en labs futuros

| Lab | Qué usa de este lab |
|-----|---------------------|
| Lab-02a RBAC | Grupos TN-* como principales de asignación de roles Azure |
| Lab-02b Policy | Tenant de TechNova como scope de las políticas |
| Lab-08 VMs | it-admin01 como administrador de las VMs |
| Lab-11 Monitoring | Grupos como destinatarios de alertas |

---

## Siguiente lab

[Lab 02a — RBAC: asignar roles Azure a los grupos de TechNova](../../Lab-02a-RBAC/lab-avanzado/README.md)

---

*Lab documentado por [RoadmapMultiCloud-ES](../../../../../README.md) · Licencia [CC BY-NC-SA 4.0](../../../../../LICENSE)*