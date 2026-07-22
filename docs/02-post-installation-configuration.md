# osTicket Lab — Part 2: Post-Installation Configuration

With osTicket installed (see [Part 1: Environment Setup & Installation](./01-environment-setup.md)), this phase configures it to reflect a real organizational structure — distinct roles, departments, teams, agents, customers, SLAs, and help topics.

> No screenshots are included — steps are documented in enough detail to be reproduced without them.

---

## Table of Contents

- [Objective](#objective)
- [Access Points](#access-points)
- [Steps](#steps)
- [Skills Demonstrated](#skills-demonstrated)
- [Lessons Learned](#lessons-learned)

---

## Objective

Configure a freshly installed osTicket instance to behave like a working IT help desk for a simulated organization, covering role-based permissions, department/team structure, agent and customer accounts, SLA response targets, and ticket intake categories.

## Access Points

| Purpose | URL |
|---|---|
| Admin / Agent Login | `http://localhost/osTicket/scp/login.php` |
| End User Portal | `http://localhost/osTicket/` |

**Agent Panel vs Admin Panel** — osTicket separates day-to-day ticket work (Agent Panel) from system configuration (Admin Panel). Agents work tickets and manage customers from the Agent Panel; admins configure roles, departments, SLAs, and topics from the Admin Panel.

---

## Steps

### 1. Roles
**Admin Panel → Agents → Roles**

Roles group permissions that get assigned to agents. Created:
- **Supreme Admin** — full administrative permissions

### 2. Departments
**Admin Panel → Agents → Departments**

Departments control ticket visibility and routing (e.g., Help Desk vs. SysAdmins vs. Networking). Created:
- **SysAdmins**

### 3. Teams
**Admin Panel → Agents → Teams**

Teams pull agents from different departments into a single working group. Created:
- **Online Banking**

### 4. Ticket Creation Policy
**Admin Panel → Settings → User Settings**

Locked down ticket creation so it's not wide open to anonymous submissions:
- **Unchecked** "Anyone can open a ticket"
- **Enabled**: *Registration Required* — users must register and log in to create tickets

### 5. Agents (Workers)
**Admin Panel → Agents → Add New**

| Agent | Department |
|---|---|
| Jane | SysAdmins |
| John | Support |

### 6. Users (Customers)
**Agent Panel → Users → Add New**

- Karen
- Ken

### 7. SLA Plans
**Admin Panel → Manage → SLA**

Defined response-time expectations by severity:

| SLA | Grace Period | Schedule |
|---|---|---|
| Sev-A | 1 hour | 24/7 |
| Sev-B | 4 hours | 24/7 |
| Sev-C | 8 hours | Business Hours |

### 8. Help Topics
**Admin Panel → Manage → Help Topics**

Categories presented to end users when they open a ticket:
- Business Critical Outage
- Personal Computer Issues
- Equipment Request
- Password Reset
- Other

---

## Skills Demonstrated

- Designing a help desk's operational structure: RBAC-style roles, departments, teams, SLAs, and intake categories
- Modeling a basic ITSM (IT Service Management) ticketing workflow end-to-end
- Applying least-privilege thinking to ticket creation policy (requiring registration instead of open/anonymous submission)
- Structuring agent/department/team relationships to support realistic ticket routing

## Lessons Learned

- Structuring the help desk (roles → departments → teams → agents) *before* adding users makes ticket routing and permissions far easier to reason about than configuring it after the fact.
- Separating SLA severity tiers by both grace period *and* schedule (24/7 vs. business hours) is what actually makes an SLA meaningful — a tight grace period on a "business hours only" schedule behaves very differently than the same grace period applied 24/7.
- Requiring registration before ticket creation is a small toggle but a meaningful control — it ties every ticket to an identifiable user rather than allowing anonymous submissions.

---

*Back to [Part 1: Environment Setup & Installation](./01-environment-setup.md).*
