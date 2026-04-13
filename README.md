# 🖥️ Microsoft Intune Home Lab

A fully functional enterprise-grade endpoint management lab built using **Microsoft Intune**, **Azure AD (Entra ID)**, and **Windows Autopilot**. This lab simulates a real-world Microsoft 365 environment and was designed to demonstrate hands-on skills in modern device management, security policy enforcement, and zero-touch provisioning.

---

## 🧰 Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Intune | MDM/MAM device and app management |
| Azure AD / Entra ID | Identity and access management |
| Windows Autopilot | Zero-touch device provisioning |
| Hyper-V | Virtual machine hosting |
| Windows 11 Pro | Enrolled Windows client |
| Android (BYOD) | Work Profile enrollment |
| Managed Google Play | Android app deployment |
| Microsoft Defender | Endpoint security |
| PowerShell | Autopilot hardware hash extraction |

---

## 👥 Tenant Structure

### Users
| Role | Type |
|---|---|
| Global Admin | Administrator |
| Intune Admin | Administrator |
| Service Desk Admin | Administrator |
| Adult Family Users | Standard Members |
| Kid Family Users | Standard Members |
| Guest User | External Guest |

### Groups

**Dynamic Device Groups** — automatically populated based on membership rules:
| Group | Rule |
|---|---|
| Windows Devices | `device.deviceOSType -eq "Windows"` |
| Android Enterprise Devices | `device.deviceOSType -eq "AndroidEnterprise"` |
| iOS Devices | `device.deviceOSType -eq "iOS"` |
| macOS Devices | `device.deviceOSType -eq "MacOS"` |
| Corporate Devices | `device.deviceOwnership -eq "Company"` |
| Personal Devices | `device.deviceOwnership -eq "Personal"` |

**Static User Groups** — manually assigned members:
- Admin Users
- Adult Users
- Kid Users
- Guest Users
- Lab Autopilot Devices *(scoped group for Autopilot profile assignment)*

---

## 📱 Device Enrollment

| Device | Type | Enrollment Method |
|---|---|---|
| Windows 11 VM (x2) | Corporate | Windows Autopilot |
| Android Phone (x2) | BYOD | Android Enterprise Work Profile |

---

## 🚀 Windows Autopilot Configuration

Zero-touch provisioning was configured end-to-end:

- **Hardware hash** extracted via PowerShell (`Get-WindowsAutoPilotInfo`) and uploaded to Intune
- **Autopilot Deployment Profile** configured with:
  - Deployment mode: User-Driven
  - Join type: Azure AD Joined
  - User account type: Standard (non-admin)
- **Enrollment Status Page (ESP)** configured to:
  - Block device use until all apps and profiles are installed
  - Show installation progress during OOBE
  - Allow device reset on error
- Devices automatically provisioned as **Corporate Owned** upon Autopilot enrollment

---

## 📋 Policies & Configuration

### ✅ Compliance Policy
- Requires BitLocker encryption
- Requires minimum OS version
- Noncompliant devices trigger automated email notification

### 🔄 Windows Update Ring
- Feature updates deferred: **7 days**
- Quality updates deferred: **7 days**
- Restarts enforced outside business hours only

### ⚙️ Configuration Profile — `My Lab Config Policy`
Applied to all Windows devices via dynamic group assignment:

| Category | Setting | Value |
|---|---|---|
| LAPS | Enable local admin password management | Enabled |
| BitLocker | Require Device Encryption | Enabled |
| Defender | Allow Archive Scanning | Enabled |
| Defender | Allow Email Scanning | Enabled |
| Defender | Allow Full Scan on Mapped Network Drives | Enabled |
| Defender | Allow Full Scan Removable Drive Scanning | Enabled |
| Defender | Allow Script Scanning | Enabled |
| Device Lock | Enforce Lock Screen and Logon Image | Company Devices |
| Security | Require Device Encryption | Encryption is required |
| Windows Hello for Business | Use Windows Hello for Business | True |
| Windows Hello for Business | PIN Expiration | 0 (never expires) |

### 🔐 Security Baseline
- Applied **Windows 10/11 Security Baseline** to lab devices
- Reviewed conflict reports and resolved policy overlaps

### 📲 Android App Protection Policy (MAM)
Applied to BYOD Android devices without full device enrollment:

- Screenshots: **Blocked**
- PIN required for app access: **Enabled**
- Copy/paste between work and personal apps: **Blocked**
- Managed browser required for web links: **Enabled**

---

## 📦 Application Deployment

### Android
| App | Deployment Type | Status |
|---|---|---|
| Required Work App (MFA) | Required | ✅ Successfully deployed |
| Managed Google Play Apps (x4) | Available | ✅ Available in Work Profile |

### Windows
| App | Deployment Type | Status |
|---|---|---|
| Required Windows App | Required | ✅ Successfully deployed |
| Spotify | Available | ✅ Available via Company Portal |
| ChatGPT | Available | ✅ Available via Company Portal |

---

## 🔒 Security Features Validated

### LAPS (Local Administrator Password Solution)
- Enabled at tenant level in Entra ID
- Local administrator account enabled via Settings Catalog profile
- Unique password automatically generated per device
- Password visible and recoverable under **Devices → Local Admin Password** in Intune
- Password rotation supported via Intune remote action

### BitLocker
- Device encryption enforced via Configuration Profile
- Recovery key automatically escrowed to **Azure AD / Entra ID**
- Encryption status verified via `manage-bde -status` on enrolled VM

### Windows Hello for Business
- Enforced on all enrolled Windows devices
- Users prompted to set up PIN on first sign-in
- PIN expiration set to never (enterprise standard)

### Microsoft Defender
- Archive, email, network drive, removable drive, and script scanning all enabled
- Defender status monitored via Intune reports dashboard

---

## 📊 Reports & Monitoring

The following reports were reviewed and documented:

- Device Compliance Report
- Compliance Status Report
- Configuration Profile Assignment Status
- Defender Agent Status Report
- Firewall Status Report
- Device Attestation Status Report
- Reports Dashboard Overview

---

## 🏗️ Lab Architecture

```
Microsoft Intune Tenant
│
├── Entra ID (Azure AD)
│   ├── Users (6 accounts across 4 roles)
│   └── Groups (10 dynamic + static groups)
│
├── Windows Devices
│   ├── VM 1 — Autopilot enrolled, compliance + config policies applied
│   └── VM 2 — Autopilot enrolled, standard user, policy validated
│
├── Android Devices
│   ├── Android BYOD #1 — Work Profile enrolled
│   └── Android BYOD #2 — Work Profile enrolled
│
└── Policies
    ├── Compliance Policy
    ├── Configuration Profile
    ├── Update Ring
    ├── Security Baseline
    ├── App Protection Policy (Android)
    └── LAPS Policy
```

---

## 💡 Key Concepts Demonstrated

- **Zero-Touch Provisioning** via Windows Autopilot — devices provisioned without IT physically touching them
- **MAM without MDM** — protecting work data on personal Android devices without enrolling the full device
- **Dynamic Group Scoping** — policies automatically apply to correct devices based on OS type and ownership
- **Least Privilege** — standard user accounts enforced via Autopilot deployment profile
- **Corporate vs BYOD separation** — different policy sets applied based on device ownership type
- **LAPS** — unique local admin passwords per device, centrally managed and recoverable
- **Compliance-driven access** — noncompliant devices trigger automated notifications

---

## 🛠️ Tools & Setup

- **Hypervisor:** Microsoft Hyper-V (Windows 10 Pro host)
- **VM OS:** Windows 11 Pro (Generation 2, Virtual TPM enabled)
- **Tenant:** Microsoft Intune free trial
- **Hash Extraction:** `Get-WindowsAutoPilotInfo` PowerShell script
- **Mobile Devices:** Physical Android devices enrolled via Work Profile

---

## 📁 Repository Structure

```
Intune-HomeLab/
│
├── README.md
├── Users-and-Groups/
├── Device-Enrollment/
├── Autopilot-and-ESP/
├── Policies-and-Profiles/
├── App-Deployment/
├── Security/
└── Reports/
```

---
