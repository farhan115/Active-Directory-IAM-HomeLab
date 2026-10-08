# Lab 03 — Active Directory Organizational Unit Management

## Project Overview

This hands-on lab demonstrates how to create, organize, and verify Organizational Units (OUs) in Microsoft Active Directory using Windows Server 2025, Windows PowerShell, and Active Directory Users and Computers (ADUC).

The purpose of this project is to develop practical Identity and Access Management (IAM) and Windows Server administration skills.

**Lab Status:** Completed — Successfully Verified

## Lab Environment

| Component | Configuration |
|---|---|
| Virtualization | VMware Workstation |
| Operating System | Windows Server 2025 |
| Domain Controller | Win2K25-DC01 |
| Active Directory Domain | farhanlab.local |
| Domain Functional Level | Windows2025Domain |
| Tools | Windows PowerShell, ADUC |
| CPU | 2 vCPU |
| Memory | 4 GB |
| Network | NAT |

## Lab Objectives

1. Understand the purpose of Organizational Units.
2. Create a parent OU for IAM lab exercises.
3. Create departmental OUs for IT, HR, and Sales.
4. Use PowerShell to manage Active Directory objects.
5. Verify the OU hierarchy through PowerShell and ADUC.
6. Troubleshoot duplicate-object errors.

## Active Directory OU Structure

```text
farhanlab.local
│
└── IAM-Lab03
    ├── IT
    ├── HR
    └── Sales
```

## Step 1 — Verify the Active Directory Domain

Opened Windows PowerShell with administrative privileges.

```powershell
Get-ADDomain | Select-Object DNSRoot
```

Expected domain:

`farhanlab.local`

## Step 2 — Create the Parent Organizational Unit

```powershell
New-ADOrganizationalUnit `
    -Name "IAM-Lab03" `
    -Path "DC=farhanlab,DC=local" `
    -ProtectedFromAccidentalDeletion $true
```

This command creates a dedicated OU for the lab and enables accidental deletion protection.

The parent OU was subsequently confirmed to exist.

## Step 3 — Create Departmental OUs

Defined the parent OU path:

```powershell
$ParentOU = "OU=IAM-Lab03,DC=farhanlab,DC=local"
```

Created departmental Organizational Units:

```powershell
"IT","HR","Sales" | ForEach-Object {
    New-ADOrganizationalUnit `
        -Name $_ `
        -Path $ParentOU `
        -ProtectedFromAccidentalDeletion $true
}
```

The resulting structure contains three departmental OUs: IT, HR, and Sales.

## Step 4 — Troubleshooting

While rerunning the OU creation command, PowerShell returned:

```text
New-ADOrganizationalUnit:
An attempt was made to add an object
to the directory with a name that
is already in use.
```

**Root cause:** The departmental Organizational Units already existed in the specified parent OU.

**Resolution:** Instead of recreating or deleting existing objects, verified their presence using PowerShell and ADUC.

**Lesson learned:** Always check whether an Active Directory object exists before attempting to create it. Repeated provisioning commands should ideally handle existing objects gracefully.

## Step 5 — Verify Using PowerShell

Executed:

```powershell
Get-ADOrganizationalUnit -Filter * -SearchBase "OU=IAM-Lab03,DC=farhanlab,DC=local" | Select-Object Name,DistinguishedName
```

**Actual verified results:**

| Name | Distinguished Name |
|---|---|
| IAM-Lab03 | OU=IAM-Lab03,DC=farhanlab,DC=local |
| IT | OU=IT,OU=IAM-Lab03,DC=farhanlab,DC=local |
| HR | OU=HR,OU=IAM-Lab03,DC=farhanlab,DC=local |
| Sales | OU=Sales,OU=IAM-Lab03,DC=farhanlab,DC=local |

All four Organizational Units were returned successfully.

## Step 6 — Verify Using ADUC

Opened:

**Server Manager → Tools → Active Directory Users and Computers**

Expanded the domain `farhanlab.local` and selected the `IAM-Lab03` OU.

Confirmed that the IT, HR, and Sales departmental OUs appeared in the directory hierarchy.

## Screenshots and Evidence

### Screenshot 1 — ADUC Organizational Unit Structure

![Active Directory OU Structure](screenshots/lab03-aduc-ou-structure.png)

### Screenshot 2 — PowerShell OU Verification

![PowerShell OU Verification](screenshots/lab03-powershell-ou-verification.png)

## IAM Concepts Learned

- **Organizational Units:** Organize Active Directory users, computers, and other objects.
- **Distinguished Names:** Identify the location of objects within the directory hierarchy.
- **PowerShell Automation:** Create and query directory objects efficiently.
- **Accidental Deletion Protection:** Helps prevent unintended removal of directory objects.
- **Administrative Delegation:** OUs can be used to delegate administration to specific teams.
- **Troubleshooting:** Recognize duplicate-object errors and validate existing resources.

## Final Outcome

Successfully verified the following Active Directory structure:

- Parent OU: IAM-Lab03
- Child OU: IT
- Child OU: HR
- Child OU: Sales

The OU hierarchy was confirmed using both Windows PowerShell and Active Directory Users and Computers.

**Lab Result: PASS**

## Next Lab

**Lab 04 — Active Directory User Provisioning and Security Group Management**

Planned activities include creating employee accounts, assigning departmental security groups, verifying group memberships, and practicing IAM access administration.
