This is the final, hardened, and 2026-ready Markdown version of **MSA Lab 01**. I have formatted this specifically for GitHub Pages (Jekyll/Kramdown), ensuring all technical hurdles, "ghost" group remediation, and the "Day 1 Floodgate" logic are documented for your repository.

---

# MSA Lab 01: Group-Based Licensing & Data Governance

**Phase 1: The Handshake** | **Month 1** **Blueprint:** Meta Startup Acquisition (MSA) 18-Month Integration

**Environment:** 2026 Entra ID Modern Workflow (Post-Deprecation)

## 1. Objective

To establish a "Day 1 Readiness" framework by cleansing the tenant of legacy data, resolving UPN collisions from failed bulk-uploads, and provisioning a 20-user "Startup Squad" with inherited **Entra ID P2** licensing and **Administrative Unit** containment.

---

## 2. Implementation Steps

### Step 1: Environmental "Nuclear" Cleanup

Before integrating a new acquisition, legacy "ghost" data must be purged to prevent UPN (User Principal Name) collisions.

1. **Bulk Delete Legacy Users:** Navigate to **Identity > Users > All Users**. Select the failed 500-user batch and click **Delete**.
2. **Delete "Ghost" Groups:** Navigate to **Groups > All Groups**. Locate the legacy **Student** group (Assigned) and click **Delete**.
3. **Permanent Purge (Cloud Shell):** UPNs remain "reserved" in the Recycle Bin for 30 days unless manually purged.
```powershell
# Connect to Graph
Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"

# Purge the 'Deleted Users' bin
$DeletedUsers = Get-MgDirectoryDeletedItemAsUser -All
foreach ($User in $DeletedUsers) {
    Remove-MgDirectoryDeletedItem -DirectoryObjectId $User.Id
    Write-Host "Purged: $($User.UserPrincipalName)" -ForegroundColor Cyan
}

```



### Step 2: Infrastructure Setup

1. **Create Licensing Group:** **Groups > New group**.
* **Name:** `L-MSA-P2-Standard`
* **Role-assignable:** **Yes** (Protects the group from non-privileged tampering).
* **Membership:** **Assigned**.


2. **Create Administrative Unit (AU):** **Roles & admins > Admin units > Add**.
* **Name:** `Startup-AU`.


3. **License Assignment:** Go to **https://www.google.com/search?q=admin.microsoft.com > Billing > Licenses**. Assign **Entra ID P2** to the `L-MSA-P2-Standard` group.

### Step 3: Configure Usage Location (Critical)

Licenses cannot be assigned to identities without a country code. This is a common point of failure in global M&A.

1. Go to **Users > All users** and select the 20 Alpha squad users.
2. Click **Properties > Edit**.
3. Under **Settings**, set **Usage location** to **United States** (or the target region).
4. Click **Save**.

### Step 4: The Identity Handshake

1. **Group Sync:** Add the 20 Alpha users to `L-MSA-P2-Standard`.
2. **AU Sync:** Add the 20 Alpha users to `Startup-AU`.
3. **Verification:** Navigate to **User profile > Licenses**.
* **Validation:** Assignment state must show **Inherited (L-MSA-P2-Standard)**.



### Step 5: Audit Log Verification (2026 Experience)

Verify the "Actor" who initiated the membership change to ensure audit compliance.

1. Go to **Identity > Monitoring & health > Audit logs**.
2. Click the **Add member to group** event.
3. In the **Activity Details** blade, verify the **Initiated by (Actor)** section.

---

## 3. MSA Expert Q&A

| Question | Answer |
| --- | --- |
| **Why assign P2 licenses to all users?** | M&A "Day 1" requires **Conditional Access** and **PIM** for immediate security. 20 users fit within the trial/Free tier cap. |
| **Which month is "Day 1 Floodgate"?** | **Month 1.** This is when we establish the initial identity perimeter. |
| **Should Entra roles be assigned to the group?** | **Yes.** It ensures the licensing group is a "Protected Object" manageable only by Privileged Admins. |
| **Does AU membership provide Licenses?** | **No.** AUs scope administrative power; Groups grant user access/licenses. |

---

## 4. Troubleshooting Archive

### The Technical "Scar Tissue"

* **The Apple Numbers Trap:** Opening CSVs in Numbers/Excel adds hidden metadata that breaks PowerShell imports. Use **VS Code** or raw **PowerShell arrays**.
* **The "Chris" Row:** Microsoft’s CSV template contains a sample row (Row 3). It must be **deleted**, not just cleared, to avoid "Object already exists" errors.
* **SDK v2.x Parameter War:** The 2026 SDK has deprecated `-MemberId`. Always use **`-DirectoryObjectId`** for membership additions.
* **License Skipping:** If a user is in the group but has no license, check the **Usage Location**. Reprocess the license via the Group > Licenses tab.

---

**Next Step for Repository:** **Lab 01 is Complete.** Would you like me to generate the **Month 2: Conditional Access (CA) Baselines** documentation now?
