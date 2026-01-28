---
layout: default
title: "Lab 0: Meta Startup Acquisition (MSA)"
nav_order: 1
---

# Lab 0: Infrastructure & Identity Seeding
**Environment:** Microsoft Entra ID (2026 Workflow)  
**License:** P2 Trial  
**Architecture:** Single-Tenant with Logical Segmentation (Administrative Units)

---

## 1. Project Context & Strategy Pivot

### The "Double Tenant" Challenge
Initially, we planned to simulate the acquisition using two distinct Entra tenants. However, due to 2026 security restrictions on trial account creation (requiring unique hardware/phone/credit card identifiers), we pivoted to a **Single-Tenant Segmentation** model.

### The Solution: Administrative Units (AU)
To simulate the "Startup" as a separate entity, we utilize **Administrative Units**. This allows for:
* **Scoped Administration:** Assigning IT managers who can only see Startup users.
* **Identity Isolation:** Keeping the 500 acquired identities logically separated from the parent "Meta" directory.

---

## 2. Troubleshooting & Lessons Learned

### Okta Sync & Deletion Persistence
We investigated syncing from Okta but found that "Soft Deletes" in Okta leave "Ghost Identities" in Entra ID for 30 days. To ensure a clean lab environment, we moved to **Cloud-Only** identities managed via PowerShell.

### CSV Formatting Pitfalls
Initial attempts at "Bulk Upload" via the Entra Portal failed due to:
* **Rigid Headers:** Missing optional columns causing validation errors.
* **Formatting:** Invisible trailing spaces in "Department" names breaking future Dynamic Group rules.
* **Scale:** The portal interface is inefficient for 500+ users with specific AU requirements.

---

## 3. Implementation: The Master Seed Script
This script uses the **Microsoft Graph PowerShell SDK (2026 Standard)**. It automates the creation of 500 users and their immediate assignment to the `Startup-Acquisition` Administrative Unit.

```powershell
# --- CONFIGURATION ---
$domain = "yourtenant.onmicrosoft.com" 
$auName = "Startup-Acquisition"

# --- AUTHENTICATION ---
# Ensure you use these specific scopes to avoid 403 Forbidden errors
# Connect-MgGraph -Scopes "User.ReadWrite.All", "AdministrativeUnit.ReadWrite.All"

# --- EXECUTION ---
$targetAU = Get-MgDirectoryAdministrativeUnit -Filter "DisplayName eq '$auName'"
$auId = $targetAU.Id
$depts = @("Engineering", "Product", "Sales", "Marketing", "Customer Success", "HR")
$passwordProfile = @{ Password = "LabPassword2026!"; ForceChangePasswordNextSignIn = $false }

Write-Host "Seeding 500 users into AU: $auName..." -ForegroundColor Cyan

foreach ($i in 1..500) {
    $num = $i.ToString("000")
    $dept = $depts[($i-1) % $depts.Count]

    $userParams = @{
        DisplayName       = "MSA User $num"
        UserPrincipalName = "msa$num@$domain"
        MailNickname      = "msa$num"
        Department        = $dept
        AccountEnabled    = $true
        PasswordProfile   = $passwordProfile
    }

    # Create User and Assign to AU via REST Reference
    $newUser = New-MgUser -BodyParameter $userParams
    $memberUrl = "[https://graph.microsoft.com/v1.0/directory/administrativeUnits/$auId/members/](https://graph.microsoft.com/v1.0/directory/administrativeUnits/$auId/members/)`$ref"
    $memberBody = @{ "@odata.id" = "[https://graph.microsoft.com/v1.0/users/$($newUser.Id](https://graph.microsoft.com/v1.0/users/$($newUser.Id))" }
    
    Invoke-MgGraphRequest -Method POST -Uri $memberUrl -Body $memberBody
    
    if ($i % 50 -eq 0) { Write-Host "Batch Complete: $i / 500" }
}
