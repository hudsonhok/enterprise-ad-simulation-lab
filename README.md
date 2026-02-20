# Active Directory Home Lab — Virtualized Enterprise Domain Environment

## Overview

This project simulates an enterprise-grade Active Directory environment built entirely on consumer hardware using nested virtualization. The goal was to design, deploy, and troubleshoot a realistic domain infrastructure to mirror real-world IT support and sysadmin scenarios — covering identity management, policy enforcement, DNS/DHCP redundancy, and authentication workflows.

---

## Architecture

```
Physical Host (Proxmox)
└── Windows Server 2025 VM (Hyper-V Host)
    ├── VS01 — Internal Virtual Switch
    ├── DC01 — Primary Domain Controller
    ├── DC02 — Secondary Domain Controller
    └── PC01 — Windows 11 Pro Client
```

**Domain:** `int.abc.com`  
**NetBIOS Name:** `ABC`  
**Network:** Internal only (VS01 virtual switch)

---

## Infrastructure Components

### Physical Host — Proxmox
- Ran a Windows Server 2025 VM as the Hyper-V host
- The host's sole responsibility is reliable VM execution — no AD/DNS/DHCP roles installed at this layer

### Hyper-V Host — Windows Server 2025
- **Role installed:** Hyper-V
- Hosts the internal virtual switch (`VS01`) connecting all VMs
- No domain, no directory services — purely a virtualization layer

### DC01 — Primary Domain Controller
- **Roles:** Active Directory Domain Services (AD DS), DNS, DHCP
- Stood up a new forest with root domain `int.abc.com`
  - Subdomain prefix (`int.`) used to avoid split-brain DNS with a potential public `abc.com` zone
- Configured static IP
- Created the initial DHCP scope (range: `.50–.200`, gateway: `.254`)
- Set up Organizational Units (OUs), user accounts, and Group Policy Objects (GPOs)

### DC02 — Secondary Domain Controller
- **Roles:** AD DS, DNS, DHCP
- Joined the existing `int.abc.com` domain and promoted to DC
- Replicated AD from DC01 (DNS auto-configured by AD during promotion)
- Configured DHCP failover with DC01 for high availability
- Provides DNS redundancy — clients have two authoritative DNS servers

### PC01 — Windows 11 Pro Client
- Joined to `int.abc.com` domain
- Used to validate and test the full environment end-to-end

---

## What Was Tested & Validated

| Scenario | Details |
|---|---|
| **DNS Resolution** | Confirmed name-to-IP resolution via both DC01 and DC02 |
| **DHCP Lease Assignment** | PC01 received IP, gateway, and DNS from the DHCP scope with failover active |
| **Domain Login / Kerberos Auth** | Logged into PC01 with domain credentials; Kerberos tickets issued correctly |
| **Password Reset** | Tested admin-initiated password resets and first-login change prompts |
| **Group Policy** | Applied and verified GPOs (password policies, desktop restrictions, etc.) |
| **AD Replication** | Confirmed changes on DC01 propagated to DC02 |
| **DHCP Failover** | Verified lease continuity when one DC was taken offline |

---

## Key Concepts

**Why `int.abc.com` instead of `abc.com`?**  
Using a subdomain for the internal AD forest prevents split-brain DNS — a scenario where internal machines can't resolve the domain correctly because it conflicts with a public DNS zone. The `int.` prefix keeps internal and external DNS cleanly separated.

**Why two domain controllers?**  
A single DC is a single point of failure. With DC02, the environment has:
- Continued Kerberos authentication if one DC goes down
- DNS redundancy (clients query either DC)
- DHCP failover (no lease gaps on DC failure)
- AD replication (changes sync between both DCs)

**Why does AD depend so heavily on DNS?**  
Clients locate domain controllers through DNS SRV records. Without working DNS, there's no way to find a DC — and without a DC, no one can log in. Every part of AD (replication, Kerberos, Group Policy, LDAP) relies on DNS being correct and consistent across both DCs.

---

## Screenshots

Documentation captured at each major milestone:

- Windows Server 2025 VM post-install
- Hyper-V role installation wizard
- AD DS installation wizard on DC01
- First domain login (`ABC\<user>`)
- OU structure creation on DC01
- DC02 joining the domain
- DHCP failover configuration

---

## Skills Demonstrated

- Nested virtualization (Proxmox → Hyper-V)
- Active Directory forest/domain deployment
- Domain controller promotion and AD replication
- DNS configuration and redundancy
- DHCP scope setup and failover
- Group Policy Object (GPO) creation and enforcement
- User provisioning, password policy, and account management
- Authentication troubleshooting (Kerberos, domain join, SRV records)
