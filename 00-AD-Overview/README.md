#=> Active Directory Overview

## What is Active Directory?

**Active Directory (AD)** is a directory service for Windows network environments. It is a distributed, hierarchical structure that allows centralized management of an organization's resources, including users, computers, groups, network devices, file shares, group policies, devices, and trusts.

A directory service such as [Active Directory Domain Services (AD DS)] gives an organization a way to store directory data and make it available to both standard users and administrators on the same network. AD DS stores information such as usernames and passwords, and manages the rights needed for authorized users to access this information.

---

## Key Terms Refresher

To make sure we are familiar with the basic concepts, here is a quick review of the main terms.

### Structure

| Term | Description |
|---|---|
| **Domain** | A group of objects (such as users or devices) that share the same AD database. |
| **Tree** | A collection of AD domains that begins at a single root domain. Each domain in a tree shares a boundary with the other domains. A parent-child trust relationship is formed when a domain is added under another domain in a tree. |
| **Forest** | A collection of AD trees (and therefore of AD domains). |
| **Organizational Unit (OU)** | A container used to store similar objects for ease of administration. OUs are often used for delegating administrative tasks without granting a user full administrative rights. |
| **Group Policy Object (GPO)** | A virtual collection of policy settings. Each GPO has a unique GUID. It can contain local file system settings or AD settings, and can be applied to both user and computer objects. |

### Identifiers and Naming

| Term | Description |
|---|---|
| **GUID** (Global Unique Identifier) | A unique 128-bit value assigned to every object AD creates (users, groups, computers, etc.). It is unique across the enterprise, similar to a MAC address, and is stored in the `ObjectGUID` attribute. |
| **Security Principal** | Any entity the operating system can authenticate: user accounts, computer accounts, security groups, or even threads/processes running in the context of a user or computer account. |
| **SID** (Security Identifier) | A unique identifier for a security principal or security group. Every account, group, or process has its own SID, issued by the domain controller and stored in a secure database. A SID can only be used once. |
| **DN** (Distinguished Name) | The full path to an object in AD. Must be unique in the directory. Example: `cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local` |
| **RDN** (Relative Distinguished Name) | A single component of the DN that identifies the object as unique at its current level. Must be unique within an OU. In the example above, `bjones` is the RDN. `cn=bjones,dc=dev,dc=inlanefreight,dc=local` is different from `cn=bjones,dc=inlanefreight,dc=local`. |
| **sAMAccountName** | The user's logon name (for example `bjones`). Must be unique and 20 characters or fewer. |
| **userPrincipalName (UPN)** | Another way to identify users, in the format `bjones@inlanefreight.local`: a prefix (account name) plus a suffix (domain name). This attribute is not mandatory. |
| **FQDN** (Fully Qualified Domain Name) | The complete name of a specific computer or host, in the format `hostname.domainname.tld`. Example: `DC01.INLANEFREIGHT.LOCAL` |
| **SPN** (Service Principal Name) | Uniquely identifies a service instance. Kerberos uses SPNs to associate a service instance with a logon account, so a client can request authentication to the service without knowing the account name. |

### Domain Controller Roles and Components

| Term | Description |
|---|---|
| **FSMO Roles** | Microsoft separated the various responsibilities that a DC can have into Flexible Single Master Operation (FSMO) roles. These give Domain Controllers (DC) the ability to continue authenticating users and granting permissions without interruption (authorization and authentication). There are five FSMO roles: Schema Master and Domain Naming Master (one of each per forest), Relative ID (RID) Master (one per domain), Primary Domain Controller (PDC) Emulator (one per domain), and Infrastructure Master (one per domain).  These roles help replication in AD to run smoothly and ensure that critical services are operating correctly. |
| **Global Catalog (GC)** | A domain controller that stores a full copy of all objects in its own domain and a partial copy of objects from other domains in the forest. |
| **RODC** (Read-Only Domain Controller) | A DC with a read-only AD database. No AD account passwords are cached on it. It also includes a read-only DNS server, allows administrator role separation, reduces replication traffic, and prevents SYSVOL modifications from being replicated to other DCs. |
| **SYSVOL** | A shared directory on each domain controller. It is crucial for replicating files among all DCs in a domain, using File Replication Services (FRS). |
| **NTDS.DIT** | The "heart" of Active Directory. A database stored on the DC at `C:\Windows\NTDS\` that holds user and group objects, group memberships, and, most importantly to attackers, the password hashes of all domain users. |

---
## Why This Matters for Security

- **NTDS.DIT** contains the password hashes for the whole domain, so it is a top target after compromising a DC.
- **SPNs** are the basis of the Kerberoasting attack (covered in the next research).
- **SYSVOL** and **GPOs** can be abused for persistence or credential discovery.
- **SIDs, GUIDs, and DNs** are what you will see in logs when investigating an incident as a SOC analyst.

---

## MSRPC and Windows Management Tools

**MSRPC** is Microsoft's implementation of **Remote Procedure Call (RPC)**, an inter-process communication technique used by client-server applications. Windows systems use MSRPC to access and manage systems in Active Directory.

An **RPC interface** is a set of procedures that a client can call on a server, over the network or across process boundaries, as if they were local calls. It works like a formal contract that defines:

- Procedure names or numbers
- Input parameters
- Return types
- A unique interface identifier (such as a UUID)

### Related Management Technologies

| Technology | Description |
|---|---|
| **WMI** (Windows Management Instrumentation) | A subsystem of PowerShell that gives administrators powerful tools for system monitoring. Its goal is to consolidate device and application management across corporate networks. |
| **MMC** (Microsoft Management Console) | Groups snap-ins (administrative tools) to manage hardware, software, and network components on a Windows host. Administrators can build custom consoles containing only the tools they need, and snap-ins can manage both local and remote systems. |

### Why This Matters for Security

- MSRPC is a common channel for **remote enumeration and lateral movement** in AD environments.
- WMI and MMC are legitimate admin tools, so attackers abuse them to blend in with normal activity. This is why a SOC analyst must be able to tell normal admin use from suspicious use.

---

## Important network ports in any Windows environment include:

| Port | Service |
|---|---|
| 53 | DNS |
| 88 | Kerberos |
| 135 | WMI/RPC |
| 137-139 & 445 | SMB |
| 389 & 636 | LDAP |
| 3389 | RDB |
| 5985 & 5986 | PowerShell Remoting (WinRM) |


