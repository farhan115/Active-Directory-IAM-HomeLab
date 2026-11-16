# Lab 09 — Active Directory Service Accounts & Least Privilege

**Author:** [Farhan Muntasir](https://github.com/farhan115)  
**Completed:** November 16, 2026  
**Environment:** Windows Server AD domain-controller VM, VMware Workstation, Windows PowerShell

## Overview

Created a dedicated Active Directory service identity and granted read-only access to a practice reports directory through a domain-local security group. Verified account settings, group membership, folder Effective Access, and inherited file permissions.

**Outcome:** The lab identity had read access to the Reports folder without write or delete permissions in the Effective Access evaluation. The sample file inherited the report group's Read & Execute permission.

This README is standalone and does not require screenshot files to render.

## Business scenario

A reporting workload needs to read report files without modifying or deleting them. Instead of using an administrator account, create a dedicated identity, document its owner and purpose, and grant resource access through a narrowly scoped security group.

This exercise models the identity and permissions configuration. No actual reporting service was installed or run.

## Environment and resources

| Resource | Value |
|---|---|
| Server VM | `Win2K25-DC01` |
| AD DNS domain | `farhanlab.local` |
| NetBIOS domain | `FARHANLAB` |
| Domain distinguished name | `DC=farhanlab,DC=local` |
| Lab OU | `OU=Lab09-Service-Accounts,DC=farhanlab,DC=local` |
| Account | `svc_lab09_reports` |
| User principal name | `svc_lab09_reports@farhanlab.local` |
| Resource access group | `DL-Lab09-Reports-Read` |
| Group type and scope | Security / Domain Local |
| Reports folder | `C:\Lab09-ServiceAccount\Reports` |
| Practice file | `sample-report.txt` |

The dedicated practice folder was hosted on the domain-controller VM for this home lab. In an enterprise design, host application workloads and their data on suitable member servers rather than treating a domain controller as an application server.

## Access design

| Principal | Intended access |
|---|---|
| `svc_lab09_reports` | Receives report access through group membership |
| `FARHANLAB\DL-Lab09-Reports-Read` | Read & Execute on reports |
| `BUILTIN\Administrators` | Full control |
| `NT AUTHORITY\SYSTEM` | Full control |

Group-based access separates account management from resource permissions. This compact lab adds the account directly to a domain-local resource group. Larger deployments can use an Accounts → Global group → Domain Local group → Permissions model when appropriate.

## Prerequisites

- An existing AD domain and working domain controller.
- Windows PowerShell with the ActiveDirectory module.
- Administrative permissions for the lab OU, group, account, and new local folder.
- A unique lab password that satisfies domain policy.

Commands below assume a new lab OU, account, and group. If an object already exists, inspect it before reusing or changing it. Do not rerun account creation blindly.

## 1. Verify the machine and domain

An initial attempt on the physical Windows 11 Pro host failed because the ActiveDirectory module was unavailable. The lab continued inside the Windows Server domain-controller VM.

Diagnostic commands:

```powershell
hostname
Get-CimInstance Win32_OperatingSystem | Select-Object Caption
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

Inside the server VM:

```powershell
Import-Module ActiveDirectory

Get-ADDomain |
    Select-Object DNSRoot, NetBIOSName, DistinguishedName

Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

**Explanation:** Verify the execution environment before changing AD. The successful output established the actual domain and OU paths, avoiding assumptions about distinguished names.

## 2. Create the dedicated OU and account

```powershell
New-ADOrganizationalUnit `
    -Name "Lab09-Service-Accounts" `
    -Path "DC=farhanlab,DC=local" `
    -ProtectedFromAccidentalDeletion $true

$servicePassword = Read-Host "Enter the lab service-account password" -AsSecureString

New-ADUser `
    -Name "svc_lab09_reports" `
    -SamAccountName "svc_lab09_reports" `
    -UserPrincipalName "svc_lab09_reports@farhanlab.local" `
    -Path "OU=Lab09-Service-Accounts,DC=farhanlab,DC=local" `
    -Description "Lab09 reporting service identity - owner: Farhan Muntasir" `
    -AccountPassword $servicePassword `
    -Enabled $true `
    -PasswordNeverExpires $false `
    -ChangePasswordAtLogon $false

Remove-Variable servicePassword
```

| Setting | Explanation |
|---|---|
| ProtectedFromAccidentalDeletion | Adds protection against accidental OU deletion; it is not a backup |
| Read-Host -AsSecureString | Avoids entering the password as plaintext in the command |
| Enabled True | Allows the identity to authenticate subject to applicable policies |
| PasswordNeverExpires False | Keeps the account subject to applicable password-expiration policy |
| ChangePasswordAtLogon False | Avoids requiring a first interactive password change for the workload identity |
| Description | Records purpose and owner |

Removing the variable drops that reference from the session; it is not a guarantee that all password material has been erased from memory. Password rotation was not automated in this lab.

## 3. Verify account settings and baseline membership

```powershell
Get-ADUser svc_lab09_reports `
    -Properties Enabled, PasswordNeverExpires, Description |
    Select-Object SamAccountName, Enabled, PasswordNeverExpires,
                  DistinguishedName, Description

Get-ADPrincipalGroupMembership svc_lab09_reports |
    Select-Object Name, GroupScope
```

Observed before adding the resource group:

| Check | Result |
|---|---|
| Account enabled | True |
| PasswordNeverExpires | False |
| OU location | Lab09-Service-Accounts |
| Description | Reporting purpose and owner recorded |
| Baseline group membership | Domain Users |

No administrative group membership was added. Domain Users membership still carries the domain's normal baseline access; the identity is not globally restricted to this one folder.

## 4. Create a resource group and practice data

```powershell
New-ADGroup `
    -Name "DL-Lab09-Reports-Read" `
    -SamAccountName "DL-Lab09-Reports-Read" `
    -GroupCategory Security `
    -GroupScope DomainLocal `
    -Path "OU=Lab09-Service-Accounts,DC=farhanlab,DC=local" `
    -Description "Read-only access to the Lab09 reports folder"

Add-ADGroupMember `
    -Identity "DL-Lab09-Reports-Read" `
    -Members "svc_lab09_reports"

New-Item -Path "C:\Lab09-ServiceAccount\Reports" -ItemType Directory -Force

"Lab09 reporting data - read-only service access" |
    Set-Content "C:\Lab09-ServiceAccount\Reports\sample-report.txt"
```

**Explanation:** The Domain Local security group represents access to this resource. Assign permissions to the group and manage the identity's membership separately. The sample file provides a child object for inheritance verification.

## 5. Configure NTFS permissions

Apply these changes only to the dedicated lab folder created above.

```powershell
$reportPath = "C:\Lab09-ServiceAccount\Reports"

# Preserve system and administrative management
icacls $reportPath /grant:r '*S-1-5-18:(OI)(CI)F' '*S-1-5-32-544:(OI)(CI)F'

# Remove inherited entries from the parent
icacls $reportPath /inheritance:r

# Grant the reporting group read and traversal access
icacls $reportPath /grant:r 'FARHANLAB\DL-Lab09-Reports-Read:(OI)(CI)RX'

icacls $reportPath

Get-ADGroupMember "DL-Lab09-Reports-Read" |
    Select-Object Name, SamAccountName
```

| Symbol or option | Meaning |
|---|---|
| `S-1-5-18` | Well-known SID for SYSTEM |
| `S-1-5-32-544` | Well-known SID for built-in Administrators |
| `/grant:r` | Replaces explicit allow grants for the specified principal; it does not reset every ACL entry |
| `/inheritance:r` | Disables inheritance and removes inherited entries |
| `(OI)` | Object inheritance: files can inherit the entry |
| `(CI)` | Container inheritance: subfolders can inherit the entry |
| `F` | Full control |
| `RX` | Read & Execute, including folder traversal |
| `(I)` | Entry inherited by the displayed object |

Administrative and SYSTEM entries were established before inherited entries were removed. A pre-existing explicit Administrators entry was also visible in this run; multiple entries do not necessarily indicate an error. Inspect the complete ACL rather than expecting identical text ordering.

## 6. Evaluate effective folder access

1. Open File Explorer at `C:\Lab09-ServiceAccount`.
2. Right-click **Reports → Properties → Security → Advanced**.
3. Open **Effective Access → Select a user**.
4. Select `FARHANLAB\svc_lab09_reports`.
5. Choose **View effective access**.

The selected account was confirmed as `svc_lab09_reports@farhanlab.local`.

| Permission | Observed result |
|---|---|
| Traverse folder / execute file | Allowed |
| List folder / read data | Allowed |
| Read attributes and extended attributes | Allowed |
| Read permissions | Allowed |
| Create files / write data | Not allowed |
| Create folders / append data | Not allowed |
| Write attributes and extended attributes | Not allowed |
| Delete subfolders and files | Not allowed |
| Delete | Not allowed |
| Change permissions | Not allowed |
| Full control | Not allowed |

**Interpretation:** The displayed results support the intended read-only folder access. A permission not being allowed does not necessarily mean an explicit Deny entry exists; the ACL may simply lack an applicable Allow.

This was an Effective Access evaluation, not a live process reading and writing under the identity. Interactive logon rights were not granted to the service account on the domain controller.

## 7. Verify child-file inheritance

```powershell
icacls "C:\Lab09-ServiceAccount\Reports\sample-report.txt"
```

Observed file permissions:

```text
FARHANLAB\DL-Lab09-Reports-Read:(I)(RX)
BUILTIN\Administrators:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
```

The command reported one file processed successfully and zero failures. `(I)(RX)` confirms the report group's Read & Execute permission was inherited by the sample file.

## Verification checklist

- [x] Correct server and AD domain identified.
- [x] Dedicated OU created.
- [x] Conventional service account created with an owner description.
- [x] Account enabled and PasswordNeverExpires verified False.
- [x] No administrative group added.
- [x] Domain-local resource group created and membership verified.
- [x] Dedicated report folder and sample file created.
- [x] Group granted RX; SYSTEM and Administrators retained F.
- [x] Folder Effective Access evaluated for the selected service identity.
- [x] Sample-file inheritance confirmed.
- [ ] Actual service execution under this identity — not performed.
- [ ] Interactive-logon restriction policy — not configured.
- [ ] Automated password rotation or gMSA — not configured.
- [ ] SMB share permissions — no share created or tested.

## Explanation and interview questions

### 1. Why use a dedicated service identity?

It separates workload access from an administrator's everyday identity, supports ownership tracking, and allows permissions to be scoped to the workload's needs.

### 2. Is this a managed service account?

No. It is a conventional AD user used to model a service identity. A gMSA provides managed password behavior for compatible workloads and would be a separate implementation.

### 3. Why grant access through a group?

It makes resource permissions easier to review and lets administrators change authorized identities through membership rather than repeatedly editing the resource ACL.

### 4. Why Domain Local scope?

It is suitable for assigning permissions to resources in the group's domain. This lab used a direct user-to-resource-group assignment; an additional Global role group can be useful in a larger access model.

### 5. What is least privilege in this exercise?

The identity needs to read reports, so the report group receives Read & Execute rather than Modify or Full Control on that resource.

### 6. Is the account restricted to accessing only Reports across the domain?

No. This exercise controls permissions on one folder. Other domain and local permissions, including baseline group access, remain relevant elsewhere.

### 7. What is the difference between an ACL and Effective Access?

The ACL lists permission entries. Effective Access evaluates the selected identity against applicable permissions and group membership to show resulting access.

### 8. Why verify the file as well as the folder?

A child file may have different inheritance settings or explicit entries. Inspecting the file confirms that the intended group permission reached that object.

### 9. Does PasswordNeverExpires False automate password rotation?

No. It leaves the account subject to applicable expiration policy. A real workload needs an intentional rotation mechanism or a supported managed identity approach to avoid outages.

### 10. Does Read & Execute permit editing or deleting reports?

That grant does not provide write, modify, or delete permissions. Other applicable ACL entries could still provide them, which is why the full ACL and effective access must be inspected.

### 11. Why retain Administrators and SYSTEM Full Control?

The system and authorized administrators need to manage the dedicated resource. Removing all management access can create recovery problems.

### 12. What would you implement next?

Use a member server to test actual workload behavior, evaluate gMSA compatibility, define logon restrictions, implement credential lifecycle controls, and monitor relevant access events.

## Troubleshooting questions and answers

### 1. Why did Import-Module ActiveDirectory fail?

The initial commands ran on the Windows 11 host, where the module was unavailable. Checking hostname and operating system identified the mismatch. Running the commands inside the configured domain-controller VM resolved it. Installing tools on a host would not turn that host into the domain controller.

### 2. Why were Get-ADDomain and Get-ADOrganizationalUnit unrecognized?

They depend on the ActiveDirectory module. Diagnose the machine and module availability before changing AD settings or installing unrelated software.

### 3. What if the OU or account already exists?

Inspect the existing object and its distinguished name. Reuse it only if it belongs to this lab and its configuration is appropriate. Do not delete an existing object simply to make a creation command succeed.

### 4. What if New-ADUser rejects the password?

Use a new unique password that meets the domain's applicable policy. Enter it through the secure prompt; do not paste it into source code, screenshots, or a README. Check whether account creation partially succeeded before retrying.

### 5. Why might icacls fail to resolve the group?

Check the NetBIOS domain, group spelling, domain connectivity, and replication if multiple domain controllers exist. Verify the object with `Get-ADGroup "DL-Lab09-Reports-Read"`.

### 6. Why might the file lack the report group's inherited permission?

The file may have inheritance disabled or different explicit settings. Inspect its Advanced Security settings and ACL before changing it. For this disposable sample file, enable inheritance if it was unintentionally disabled, then verify again; do not reset ACLs broadly on unrelated data.

### 7. Why might Effective Access show write permission unexpectedly?

Another direct or group-based entry may grant it. Confirm the selected identity, inspect the complete ACL, and review group memberships. Remove only the unintended grant after identifying its source.

### 8. Does a red cross mean an explicit Deny entry exists?

No. It indicates the evaluated permission is not allowed. Missing Allow permissions can produce that result without any explicit Deny entry.

### 9. Why did the first Effective Access screenshot not prove the identity?

The selected user was outside the visible portion of the window. A second screenshot showed the account name, linking the evaluation to the intended service identity.

### 10. Why can results differ after adding group membership?

Existing logon tokens may retain earlier membership. A new authentication session may be needed for a live workload test; refreshing the evaluation also matters. Do not confuse stale sessions with incorrect directory membership.

### 11. Why might a live process still fail to read a file despite RX?

It may be using a different identity, lack traversal access along the path, or encounter other restrictions. For network access, share permissions and authentication also matter. This lab evaluated local NTFS permissions and did not test an SMB share.

### 12. Why not test by logging the account onto the domain controller?

A service identity does not need interactive domain-controller access for this exercise. Effective Access provides an ACL evaluation. A realistic execution test should use an appropriate member-server workload without broadly expanding logon rights.

### 13. Why might a service fail after password expiration?

A conventional user account requires coordinated credential updates. Expiration without updating the service's configured credential can interrupt authentication. Rotation design was outside this lab.

### 14. What if removing inheritance causes unexpected access loss?

Inspect the specific lab folder's ACL and retained management entries. Use the authorized administrator to restore the required entries. Avoid copying these commands to existing production directories without assessing their current permissions.

## Portfolio summary

Created an AD reporting identity and domain-local security group, configured read-only NTFS resource access, evaluated effective permissions for the service account, and verified inherited permissions on a sample report. Diagnosed an initial host-versus-server module error and documented the distinction between configuration validation and live workload testing.
