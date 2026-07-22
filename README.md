# osTicket Lab

A hands-on lab where I deployed **osTicket** (an open-source help desk / ticketing system) from scratch on a Windows 10 Azure VM — building the full web server stack (IIS + PHP + MySQL) and then configuring osTicket for a simulated organization with multiple departments, teams, agents, and SLAs.

This lab was completed in two phases:

1. **Environment Setup & Installation** — provisioning the VM and installing/configuring every dependency osTicket needs to run.
2. **Post-Installation Configuration** — setting up the help desk itself: roles, departments, teams, agents, end users, SLAs, and help topics.

> No screenshots are included in this repo — everything below is documented step-by-step in enough detail to be reproduced without them.

---

## Table of Contents

- [Objective](#objective)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Part 1: Environment Setup & Installation](#part-1-environment-setup--installation)
- [Part 2: Post-Installation Configuration](#part-2-post-installation-configuration)
- [Access Points](#access-points)
- [Skills Demonstrated](#skills-demonstrated)
- [Security Notes](#security-notes)
- [Lessons Learned](#lessons-learned)

---

## Objective

Simulate standing up an internal IT help desk from the ground up — the way a small IT/SysAdmin team might do it — covering:

- Provisioning a Windows server in the cloud
- Installing and wiring together a full LAMP-style stack on Windows (IIS instead of Apache)
- Installing a real-world open-source ticketing application (osTicket)
- Configuring the ticketing system for actual organizational use: departments, teams, agents, customers, SLAs, and help topics

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

## Part 1: Environment Setup & Installation

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

## Part 2: Post-Installation Configuration

With osTicket installed, I configured it to reflect a real organizational structure with distinct roles, departments, and workflows.

### 1. Roles (Admin Panel → Agents → Roles)
Configured permission groupings for agents. Created:
- **Supreme Admin** — full administrative permissions

### 2. Departments (Admin Panel → Agents → Departments)
Departments control ticket visibility and routing. Created:
- **SysAdmins** — separate from general Help Desk / Networking queues

### 3. Teams (Admin Panel → Agents → Teams)
Teams let you pull agents from multiple departments into a single working group. Created:
- **Online Banking** — a cross-functional team

### 4. User Registration Policy (Admin Panel → Settings → User Settings)
Locked down ticket creation so it's not wide open to anonymous submissions:
- **Unchecked** "Anyone can open a ticket"
- **Enabled**: *Registration Required* — users must register and log in to create tickets

### 5. Agents (Admin Panel → Agents → Add New)
Added staff who will work tickets:

| Agent | Department |
|---|---|
| Jane | SysAdmins |
| John | Support |

### 6. End Users / Customers (Agent Panel → Users → Add New)
Added customers who will submit tickets:
- Karen
- Ken

### 7. SLA Plans (Admin Panel → Manage → SLA)
Defined response-time expectations by severity:

| SLA | Grace Period | Schedule |
|---|---|---|
| Sev-A | 1 hour | 24/7 |
| Sev-B | 4 hours | 24/7 |
| Sev-C | 8 hours | Business Hours |

### 8. Help Topics (Admin Panel → Manage → Help Topics)
Categories presented to end users when they open a ticket:
- Business Critical Outage
- Personal Computer Issues
- Equipment Request
- Password Reset
- Other

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
- Designing a help desk's operational structure: RBAC-style roles, departments, teams, SLAs, and intake categories
- Modeling a basic ITSM (IT Service Management) ticketing workflow end-to-end

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
- Structuring the help desk (roles → departments → teams → agents) *before* adding users makes ticket routing and permissions far easier to reason about than configuring it after the fact.

---
