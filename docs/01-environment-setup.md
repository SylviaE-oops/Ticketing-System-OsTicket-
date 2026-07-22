 osTicket Lab — Part 1: Environment Setup & Installation

Provisioning a Windows 10 Azure VM and building the full web stack (IIS + PHP + MySQL) needed to run **osTicket v1.15.8** from scratch.

> No screenshots are included — steps are documented in enough detail to be reproduced without them.

---

## Table of Contents

- [Objective](#objective)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Steps](#steps)
- [Access Points](#access-points)
- [Skills Demonstrated](#skills-demonstrated)
- [Security Notes](#security-notes)
- [Lessons Learned](#lessons-learned)

---

## Objective

Simulate standing up an internal help desk server from the ground up — provisioning a cloud VM and installing/configuring every dependency osTicket needs to run on Windows/IIS.

## Tech Stack

| Component | Purpose |
|---|---|
| **Microsoft Azure** | Cloud hosting for the Windows 10 VM |
| **Windows 10 (4 vCPUs)** | Guest OS for the web server |
| **IIS (Internet Information Services)** | Web server, with CGI enabled |
| **PHP Manager for IIS** | Bridges PHP to IIS |
| **URL Rewrite Module** | Required by osTicket for clean URLs/routing |
| **PHP 7.3.8 (NTS, x86)** | Runtime osTicket is built on |
| **VC++ Redistributable (x86)** | Required runtime libraries for PHP |
| **MySQL 5.5.62** | Database backend for osTicket |
| **HeidiSQL** | GUI client for managing the MySQL database |
| **osTicket v1.15.8** | The help desk / ticketing application |

## Architecture Overview

```
                     ┌─────────────────────────────┐
                     │         Azure Cloud          │
                     │                              │
                     │  ┌────────────────────────┐  │
                     │  │   osticket-vm (Win10)   │  │
                     │  │                          │  │
                     │  │   IIS (port 80)          │  │
                     │  │    └─ CGI + PHP 7.3.8    │  │
                     │  │        └─ osTicket app   │  │
                     │  │             │            │  │
                     │  │        MySQL 5.5.62      │  │
                     │  │        (osTicket DB)     │  │
                     │  └────────────────────────┘  │
                     └──────────────┬───────────────┘
                                    │ RDP
                                    ▼
                             Admin Workstation
```

---

## Steps

### 1. Provision the Azure VM
Created a Windows 10 virtual machine in Azure:

| Setting | Value |
|---|---|
| VM Name | `osticket-vm` |
| OS | Windows 10 |
| vCPUs | 4 |
| Username | `labuser` |
| Password | *(stored in a password manager — see [Security Notes](#security-notes))* |

Connected to the VM using **Remote Desktop (RDP)**.

### 2. Stage the Installation Files
Downloaded `osTicket-Installation-Files.zip` inside the VM and extracted it to the desktop. This folder contained every installer needed for the rest of the setup:

- `PHPManagerForIIS_V1.5.0.msi`
- `rewrite_amd64_en-US.msi`
- `php-7.3.8-nts-Win32-VC15-x86.zip`
- `VC_redist.x86.exe`
- `mysql-5.5.62-win32.msi`
- `osTicket-v1.15.8.zip`
- HeidiSQL installer

### 3. Install & Configure IIS
Enabled the **IIS** Windows feature with:
- World Wide Web Services → Application Development Features → **CGI** ✔️

CGI support is required so IIS can hand off requests to the PHP interpreter.

### 4. Install Supporting Components
- **PHP Manager for IIS** — lets IIS manage and register PHP handlers through its GUI.
- **URL Rewrite Module** — osTicket depends on rewrite rules for its routing/URLs.
- **PHP 7.3.8** — created `C:\PHP` and extracted the PHP NTS (Non-Thread-Safe) build there, since IIS/CGI uses non-thread-safe PHP.
- **VC++ Redistributable (x86)** — installed to satisfy PHP's runtime dependencies.
- **MySQL 5.5.62** — installed via Typical Setup, then ran the Configuration Wizard using Standard Configuration:
  - Username: `root`
  - Password: `root`

### 5. Register PHP with IIS
- Opened **IIS as Administrator**.
- In **PHP Manager**, registered PHP by pointing it to `C:\PHP\php-cgi.exe`.
- Stopped and restarted the IIS server to apply the changes.

### 6. Deploy osTicket
- Extracted `osTicket-v1.15.8.zip` and copied the `upload` folder into `C:\inetpub\wwwroot`.
- Renamed `upload` → `osTicket`.
- Restarted IIS.
- Verified the site loaded via **Sites → Default → osTicket → Browse *:80**.

### 7. Enable Required PHP Extensions
The initial browse showed missing extension warnings. Fixed via IIS → osTicket site → PHP Manager → **Enable or disable an extension**:
- ✔️ `php_imap.dll`
- ✔️ `php_intl.dll`
- ✔️ `php_opcache.dll`

Refreshed the browser to confirm the warnings cleared.

### 8. Prepare the Config File
- Renamed `include\ost-sampleconfig.php` → `include\ost-config.php`.
- Adjusted NTFS permissions on `ost-config.php` (disabled inheritance, removed all, granted **Everyone: Full Control**) so the web installer could write to it.

### 9. Run the Web-Based Installer
Continued setup from the browser:
- Set the **Helpdesk Name**
- Configured the **default support email** (the address that receives customer emails)

### 10. Create the Database
- Installed **HeidiSQL** and connected using `root` / `root`.
- Created a new database named **`osTicket`**.

### 11. Finish Installation
Back in the browser installer:
- MySQL Database: `osTicket`
- MySQL Username: `root`
- MySQL Password: `root`
- Clicked **Install Now!**

Installation completed successfully.

### 12. Post-Install Cleanup (Security Hardening)
- Deleted the `C:\inetpub\wwwroot\osTicket\setup` directory (prevents re-running the installer).
- Locked down `ost-config.php` to **Read-only** permissions (prevents tampering with the live config).

---

## Access Points

| Purpose | URL |
|---|---|
| Admin / Agent Login | `http://localhost/osTicket/scp/login.php` |
| End User Portal | `http://localhost/osTicket/` |

---

## Skills Demonstrated

- Provisioning and remotely administering a cloud VM (Azure + RDP)
- Standing up a Windows-based web stack: IIS, CGI, PHP, and URL Rewrite
- Configuring PHP for IIS via PHP Manager, including enabling required extensions
- Installing and configuring a MySQL database server and connecting it to a web application
- Deploying and hardening a real-world open-source PHP application (osTicket)
- Applying least-privilege NTFS permission changes (locking down config files post-install)

## Security Notes

This lab used simple, shared lab credentials (e.g., `root`/`root`, plaintext VM passwords) purely for speed of setup in an isolated training environment. **This is not production-safe practice.** In a real deployment:

- Credentials should be generated randomly and stored in a password manager (e.g., KeePass, Bitwarden, 1Password) or a secrets manager (e.g., Azure Key Vault).
- Database accounts should follow least privilege (a dedicated osTicket DB user, not `root`).
- The `/setup` directory removal and read-only `ost-config.php` permission are both real hardening steps and were intentionally carried out at the end of installation.
- The VM should be restricted via Network Security Group rules to only the IPs/ports actually needed (e.g., RDP restricted to a known IP).

## Lessons Learned

- Getting PHP to talk to IIS correctly (via CGI + PHP Manager) is the trickiest part of the whole stack — version mismatches (thread-safe vs non-thread-safe PHP builds) are a common failure point.
- Missing PHP extensions won't always throw hard errors — osTicket degrades gracefully and just flags what's missing, so checking the pre-install warning page carefully matters.
- File permission issues (e.g., `ost-config.php` not being writable) are one of the most common install blockers and are easy to overlook.

---

*Continue to [Part 2: Post-Installation Configuration](./02-post-installation-configuration.md).*
