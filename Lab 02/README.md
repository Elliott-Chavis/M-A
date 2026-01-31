# Lab 02: High-Assurance Identity Hardening & M&A Segmentation

**Phase:** Month 2  
**Framework:** Meta Startup Acquisition (MSA) Integration Blueprint  
**Tooling:** Modern 2026 Entra ID Workflow

---

## Phase 1: The "Digital Vault" (Safety Net)
**Objective:** Establish a non-phishable recovery path to prevent permanent lockout.

1. **Create Cloud-Only Admins:**
   - Create `admin-vault-01` and `admin-vault-02`.
   - **Role:** Global Administrator.
   - **Domain:** Must use the `[tenant].onmicrosoft.com` domain.
2. **Create Emergency Group:**
   - **Name:** `SEC-Admin-Emergency-Access`.
   - **Settings:** Set "Entra ID roles can be assigned to the group" to **Yes**.
   - **Members:** Add both vault accounts.
3. **Registration:** Log in as these users and register a physical **FIDO2 Security Key**.
4. **The Golden Rule:** This group MUST be excluded from all Conditional Access policies created in this lab.

---

## Phase 2: Pilot Security (The CEO Hardening)
**Objective:** Force phishing-resistant MFA while handling the "53003" loop.

1. **Create Pilot Group:** `SEC-Pilot-PhishResistant-MFA`. Add high-value personas (CEO, CTO, etc.).
2. **Configure CA Policy:** `[MSA] GRANT - Phish-Resistant MFA (Pilot)`.
3. **Target Resources (UI Fix):**
   - Select **Cloud apps**.
   - On the **Include** tab, select the radio button **All resources (formerly 'All cloud apps')**.
   - *Note:* Do not use manual selection to avoid the "100 apps" scrolling trap.
4. **Grant Access:** Select **Require authentication strength** > **Phishing-resistant MFA**.
5. **State:** Set to **On**.

---

## Phase 3: The Bootstrap & Dual-Method Registration
**Objective:** Enable the CEO to register both a USB key and the Authenticator App.

1. **Issue Temporary Access Pass (TAP):**
   - Since the policy is active, the CEO is blocked (Error 53003).
   - Go to `Users` > `CEO Account` > `Authentication methods`.
   - Add **Temporary Access Pass**. Copy the generated code.
2. **Dual-Key Registration:**
   - User logs in to [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) using the **TAP**.
   - **Method A (Daily):** Add **Passkey in Microsoft Authenticator**. (Uses phone biometrics).
   - **Method B (Backup):** Add **Security Key (USB)**. (Physical FIDO2 key).
3. **Result:** The user now has two Phish-Resistant methods for redundancy.

---

## Phase 4: The Kill-Switch (Legacy Auth Block)
**Objective:** Close insecure backdoors (POP/IMAP/SMTP).

1. **Name:** `[MSA] BLOCK - Legacy Authentication`.
2. **Assignments:** Include **All users**; Exclude `SEC-Admin-Emergency-Access`.
3. **Target Resources:** **All resources**.
4. **Conditions > Client apps:**
   - Configure: **Yes**.
   - **Uncheck:** Browser, Mobile apps.
   - **Check:** **Exchange ActiveSync** and **Other clients**.
5. **Grant:** **Block Access**.
6. **State:** Set to **Report-only** for monitoring.

---

## Phase 5: Administrative Units (Segmentation)
**Objective:** Create a logical sandbox for the acquired startup.

1. **Create AU:** `Roles & admins` > `Admin units` > `AU-MSA-Startup-B`.
2. **Populate:** Add 10 users from Lab 01 to the AU.
3. **Scoped Delegation:**
   - Inside the AU, go to `Roles and administrators`.
   - Assign **Helpdesk Administrator** to `startup-it-admin`.
   - **Effect:** This admin can only manage the 10 users in this AU.

---

## Phase 6: Validation & Troubleshooting

### Error Code 53003
- **Meaning:** Policy applied, but MFA requirement (Phish-Resistant) wasn't met.
- **Fix:** Ensure the user has been issued a TAP or is excluded via the Vault group.

### UI Troubleshooting: "Missing All Apps option"
- **Fix:** You are in the "Select resources" side-window. Exit that window and use the radio buttons on the main Policy configuration screen.

### Redundancy Strategy
- **USB vs App:** The user cannot use both simultaneously. However, having both registered ensures that if a phone is lost, the USB key provides immediate recovery without IT intervention.

---
