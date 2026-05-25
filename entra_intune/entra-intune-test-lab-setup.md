# Entra ID + Intune Test Lab Setup Guide

**Goal:** Minimal-cost test environment — Microsoft Entra ID, 5 users, 2–3 Windows 11 VMs (VMware), Intune for configuration profiles and app deployment.  
**Licensing:** Microsoft 365 Business Premium 30-day free trial (no credit card required).  
**Domain:** Default `.onmicrosoft.com` (no custom domain).

---

## What You Are Building

```
Microsoft Cloud (Entra ID tenant)
│
├── 5 test users (cloud-only accounts)
├── Intune MDM enrolled
│
└── Connected devices
    ├── VM-01 (Windows 11 Ent Eval) — Entra Joined + Intune enrolled
    ├── VM-02 (Windows 11 Ent Eval) — Entra Joined + Intune enrolled
    └── VM-03 (Windows 11 Ent Eval) — Entra Joined + Intune enrolled
```

**Entra ID Join** (not Hybrid Join) — the VMs become pure cloud-managed devices, no on-premises Active Directory needed. This is the simplest topology for a test lab.

---

## Licensing: What M365 Business Premium Trial Gives You

| Feature | Included |
|---|---|
| Entra ID Plan 1 (formerly Azure AD P1) | Yes |
| Microsoft Intune Plan 1 | Yes |
| Up to 25 user licenses on trial | Yes |
| Trial duration | 30 days |
| Credit card required | No (phone verification only) |

**After 30 days:** The tenant is suspended, not deleted. You have a grace period (~30 days) to export data. If you want to continue testing, you can convert to a paid plan or start a new tenant.

**Why Business Premium and not a free Entra tier?**  
The free tier does not include Intune or Entra ID P1 (which is needed for MDM auto-enrollment). Business Premium trial is the cheapest path that includes both Entra P1 and Intune.

---

## Phase 1 — Create a Microsoft Account (Personal)

This is a regular personal Microsoft account used only to start the M365 sign-up. It is separate from your tenant admin account.

1. Go to **https://account.microsoft.com**
2. Click **Create a Microsoft account**
3. Use any email (Gmail, etc.) or create a new `@outlook.com` address
4. Complete verification (phone or email code)

> **Why:** Microsoft's trial sign-up flow requires you to be logged in with a personal Microsoft account to begin. The actual admin account you will use daily is a separate account created during the next phase.

---

## Phase 2 — Sign Up for Microsoft 365 Business Premium Trial

This step creates your **Entra ID tenant** and your **Global Administrator account**.

1. Open **https://www.microsoft.com/en-us/microsoft-365/business/compare-all-plans**
2. Find **Business Premium** and click **Try free for one month**
3. Sign in with the personal Microsoft account from Phase 1
4. When prompted for your organization:
   - **Organization name:** anything (e.g., `TestLab` or your name) — this becomes part of your `.onmicrosoft.com` domain
   - **Country/Region:** select your actual country — **this cannot be changed later**
5. **Create your admin account:**
   - Username: e.g., `admin`
   - Domain: `testlab.onmicrosoft.com` (whatever name you chose)
   - Password: strong password — save this somewhere safe
6. Verify with a phone number (SMS code)
7. On the payment page — choose **Try free** / **No commitment trial** — you do NOT need to enter card details for the trial

> The account you just created (`admin@testlab.onmicrosoft.com`) is your **Global Administrator**. Write it down. This is the main account you will use.

---

## Phase 3 — Configure Entra ID

### 3.1 Access the Admin Centers

| Admin Center | URL | Purpose |
|---|---|---|
| Microsoft 365 Admin Center | https://admin.microsoft.com | Licensing, users (simplified) |
| Entra Admin Center | https://entra.microsoft.com | Identity, devices, policies |
| Intune Admin Center | https://intune.microsoft.com | MDM, device management, apps |

Sign in to all of these with your `admin@yourtenant.onmicrosoft.com` account.

---

### 3.2 Create 5 Test Users

**Where:** Entra Admin Center → **Users** → **All users** → **New user** → **Create new user**

Create each user with these settings:

| Field | Value |
|---|---|
| User principal name | `user1@yourtenant.onmicrosoft.com` (user1 through user5) |
| Display name | User One (etc.) |
| Password | Set a temporary password — user will change on first login |
| Usage location | Your country — **required before assigning licenses** |

**Assign licenses after creating each user:**  
User → **Licenses** → **Assignments** → enable **Microsoft 365 Business Premium**

> **Why assign licenses?** Intune enrollment only works for licensed users. Without a Business Premium license, the user cannot enroll a device.

**Suggested user structure for testing:**

| Username | Role | Purpose |
|---|---|---|
| user1 | Standard user | Device owner for VM-01 |
| user2 | Standard user | Device owner for VM-02 |
| user3 | Standard user | Device owner for VM-03 |
| user4 | Standard user | Testing app assignments |
| user5 | Standard user | Testing policy exclusions |

---

### 3.3 Create Security Groups

Groups are the building blocks for targeting policies and apps in Intune.

**Where:** Entra Admin Center → **Groups** → **New group**

| Group name | Type | Members | Used for |
|---|---|---|---|
| `Intune-TestDevices` | Security | (devices, added after enrollment) | Apply device config profiles |
| `Intune-TestUsers` | Security | user1–user5 | Apply user-targeted policies and apps |

**Group type:** Use **Security** (not Microsoft 365). Assigned membership (not dynamic) is simpler for a test lab.

> **Why groups?** Intune policies are assigned to groups, not individual users or devices directly. This mirrors real-world usage.

---

### 3.4 Configure MDM Auto-Enrollment

This makes devices automatically enroll into Intune when a user joins them to Entra ID.

**Where:** Entra Admin Center → **Devices** → **Device settings** → **Manage device settings**  
*(Or search for "Mobility (MDM and MAM)" in Entra)*

Alternatively:  
**Intune Admin Center** → **Devices** → **Enrollment** → **Windows** → **Windows enrollment** → **Automatic Enrollment**

Settings:

| Setting | Value |
|---|---|
| MDM user scope | **All** (covers all licensed users) |
| MDM discovery URL | Leave default (auto-populated) |
| MAM user scope | None (not needed for device management) |

> **Why "All"?** Scoping to "All" means any licensed user who joins a device to Entra will automatically get Intune enrolled. For a test lab with 5 users, this is the safest setting — no manual enrollment step needed on the device.

---

### 3.5 Entra Device Settings

**Where:** Entra Admin Center → **Devices** → **Device settings**

| Setting | Recommended value | Reason |
|---|---|---|
| Users may join devices to Entra ID | **All** | Allows any licensed user to join a device |
| Maximum number of devices per user | **5** or **Unlimited** | Allows each test user to join multiple VMs |
| Require Multi-Factor Authentication to join | **No** | Simplifies testing — enable later if you test Conditional Access |
| Microsoft Entra joined device local administrator | Your admin account | Gives you local admin on joined devices |

---

## Phase 4 — Configure Intune

### 4.1 Set Enrollment Restrictions

**Where:** Intune Admin Center → **Devices** → **Enrollment** → **Enrollment device platform restrictions**

For the default Windows restriction:

| Setting | Value |
|---|---|
| Platform status | Enabled |
| MDM | Allowed |
| Minimum OS version | 10.0.22000 (Windows 11) |
| Maximum devices per user | 5 |

> **Why set minimum OS to 22000?** That's the Windows 11 build number. Since all your VMs will be Windows 11, this keeps the restriction clean and mirrors a real policy.

---

### 4.2 Review Compliance Policy (Optional for Now)

By default, all enrolled devices are marked as **compliant** even without an explicit compliance policy. For initial testing this is fine — you can add compliance policies later once enrollment is working.

---

## Phase 5 — Prepare Windows 11 VMs in VMware

### 5.1 Download Windows 11 Evaluation ISO

1. Go to: **https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise**
2. Download the **Windows 11 Enterprise 90-day Evaluation ISO** (English or your language)
3. No key needed — the evaluation activates automatically for 90 days

> **Why Enterprise edition?** Enterprise supports Entra ID Join and Intune enrollment. Windows 11 **Home** does NOT support Entra ID Join (it's a Pro/Enterprise feature). The evaluation ISO gives you Enterprise for free for 90 days, which aligns with your trial timeline.

### 5.2 VM Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| vCPUs | 2 | 2–4 |
| RAM | 4 GB | 4–8 GB |
| Disk | 64 GB | 80 GB |
| TPM | Required by Windows 11 | Use VMware virtual TPM |
| Secure Boot | Required | Enable in VM firmware settings |

### 5.3 Create Each VM in VMware Workstation

1. **New Virtual Machine** → Typical
2. Installer disc image file: select the Windows 11 Enterprise ISO
3. Guest OS: **Windows 11 x64**
4. Name VMs clearly: `WIN11-VM01`, `WIN11-VM02`, `WIN11-VM03`
5. Disk size: 80 GB, **Store as single file**
6. Before finishing — click **Customize Hardware:**
   - RAM: 4 GB minimum
   - CPU: 2 cores
   - Enable **Virtualize Intel VT-x/EPT** (needed for TPM emulation)
7. **VM Settings → Options → Advanced:** make sure UEFI firmware is selected
8. Add a virtual TPM: **VM Settings → Add → Trusted Platform Module**

> **Why virtual TPM?** Windows 11 requires TPM 2.0. VMware Workstation supports a software TPM — you must add it manually for Windows 11 to install correctly.

### 5.4 Windows 11 Installation (for each VM)

1. Boot the VM from the ISO
2. Language, keyboard → **Next** → **Install Now**
3. Accept license agreement
4. **Custom install** → select the unallocated drive → **Next**
5. When setup completes and reaches the OOBE (first-run screen):
   - **Select your country** → Next
   - **Keyboard layout** → Next
   - On **"How would you like to set up?"** → choose **"Set up for work or school"**
   - Click **Sign in** → enter your Entra user credentials (e.g., `user1@yourtenant.onmicrosoft.com`)
   - Complete MFA if prompted
   - Follow remaining prompts — Windows will Entra Join the device automatically

> **Why "Set up for work or school" at OOBE?** This triggers an **Entra ID Join** during Windows setup, which is the cleanest enrollment path. The device becomes Entra Joined and (because MDM auto-enrollment is enabled) automatically enrolled in Intune — all in one step, no manual action needed afterward.

---

## Phase 6 — Verify Enrollment

### On the VM (after OOBE)

Open **Settings → Accounts → Access work or school**  
You should see:  
`Connected to [Your Tenant Name] Entra ID` with an **Info** button showing MDM enrollment.

### In Intune Admin Center

**Intune → Devices → All devices**  
Each enrolled VM should appear within 5–15 minutes. Status should be:
- **Managed by:** Intune
- **Ownership:** Corporate
- **Compliance:** Compliant (default)

### In Entra Admin Center

**Entra → Devices → All devices**  
Each VM should appear with:
- **Join type:** Entra Joined
- **MDM:** Microsoft Intune

---

## Phase 7 — Create Configuration Profiles

**Where:** Intune Admin Center → **Devices** → **Configuration profiles** → **Create profile**

### Example Profile 1: Basic Security Settings

| Setting | Value |
|---|---|
| Platform | Windows 10 and later |
| Profile type | Settings catalog |
| Name | `TestLab-BasicSecurity` |

In the settings catalog, add:

| Category | Setting | Value | Reason |
|---|---|---|---|
| Defender | Turn on real-time protection | Enabled | Basic AV |
| Windows Update | Defer feature updates | 0 days | Testing only |
| Experience | Allow Cortana | Blocked | Reduce noise in test lab |
| Control Panel / Personalization | Prevent enabling lock screen camera | Enabled | Minor security policy example |

**Assign to:** `Intune-TestDevices` group

> After creating and assigning, devices check in every ~8 hours automatically, or you can force sync: **Intune → Devices → [device name] → Sync**

---

### Example Profile 2: Configure Windows Update Ring

**Where:** Intune Admin Center → **Devices** → **Windows 10 and later updates** → **Update rings** → **Create profile**

| Setting | Value |
|---|---|
| Name | `TestLab-UpdateRing` |
| Servicing channel | General Availability Channel |
| Feature update deferral | 0 days (test lab — no delay needed) |
| Quality update deferral | 0 days |
| Automatic update behavior | Auto install and restart |

**Assign to:** `Intune-TestDevices`

---

## Phase 8 — App Deployment

**Where:** Intune Admin Center → **Apps** → **Windows** → **Add**

### Option A: Deploy a Microsoft Store App (simplest)

1. App type: **Microsoft Store app (new)**
2. Search for an app (e.g., `Notepad++`, `7-Zip`, `VLC`)
3. Select the app → **Next**
4. Assignment: **Required** for `Intune-TestUsers`
5. **Required** = app is pushed automatically; **Available** = user can install from Company Portal

> **Why "Required"?** For testing push deployment, "Required" is the correct choice — the app installs without user action. Use "Available" when you want users to choose whether to install.

---

### Option B: Deploy a Win32 App (.exe / .msi)

For deploying custom or legacy apps (e.g., an `.msi` installer):

1. Package the app using the **Microsoft Win32 Content Prep Tool**:
   - Download: https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool
   - Run: `IntuneWinAppUtil.exe -c <source folder> -s <setup file> -o <output folder>`
   - This creates a `.intunewin` file
2. In Intune → Apps → Windows → **Add** → **Windows app (Win32)**
3. Upload the `.intunewin` file
4. Set install command (e.g., `setup.exe /quiet /norestart`)
5. Set detection rule (registry key or file presence)
6. Assign to group

---

## Phase 9 — Testing Checklist

After completing setup, verify each of these:

- [ ] 5 users exist in Entra Admin Center with licenses assigned
- [ ] All 3 VMs appear in **Intune → Devices → All devices**
- [ ] All 3 VMs show **Entra Joined** in **Entra → Devices → All devices**
- [ ] Configuration profiles show **Succeeded** in device status (allow 15–30 min after enrollment)
- [ ] App deployment shows **Installed** on target devices
- [ ] You can log into each VM using different user credentials (`user2@...`, `user3@...`)

**Force a policy sync if waiting:**  
On the device: `Settings → Accounts → Access work or school → [connection] → Info → Sync`  
Or in Intune portal: **Devices → [device] → Sync**

---

## Cost Summary

| Item | Cost |
|---|---|
| M365 Business Premium trial (25 users, 30 days) | Free |
| Windows 11 Enterprise Evaluation (90 days) | Free |
| VMware Workstation Player (personal use) | Free |
| Entra ID / Intune (included in M365 BP) | Free |
| **Total** | **$0 for 30 days** |

**After 30 days:** The tenant suspends but is not immediately deleted. You have a grace period to decide. Options:
- Start a new trial tenant (different email address)
- Purchase M365 Business Premium (~$22/user/month) for continued use

---

## Quick Reference: Admin Center URLs

| Center | URL |
|---|---|
| M365 Admin | https://admin.microsoft.com |
| Entra (Azure AD) | https://entra.microsoft.com |
| Intune | https://intune.microsoft.com |
| Azure Portal (if needed) | https://portal.azure.com |

---

## Common Issues and Fixes

| Problem | Likely cause | Fix |
|---|---|---|
| Device won't Entra Join | "Set up for work or school" not selected at OOBE | Reset the PC (`Settings → System → Recovery → Reset this PC`) and redo OOBE |
| Intune shows device not enrolled | MDM auto-enrollment scope not set to "All" | Check Entra → Devices → Mobility (MDM and MAM) |
| Policy shows "Pending" for a long time | Device hasn't checked in | Force sync from Intune portal or from device Settings |
| App not installing | User/device group assignment incorrect | Check app assignment and that device/user is in the assigned group |
| Windows 11 won't install in VMware | Missing virtual TPM or Secure Boot | Add TPM in VM settings, ensure UEFI firmware selected |
| User can't sign in during OOBE | License not assigned | Assign M365 Business Premium license to that user in Entra before joining |

---

*Generated: 2026-05-25 | Setup scope: test/lab environment, not production*
