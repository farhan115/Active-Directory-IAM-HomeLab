# Lab 05 — Active Directory Security Group Management

## Objective
Create and manage an Active Directory security group and verify user membership using PowerShell.

## Lab Environment
- Windows Server 2025
- VMware Workstation
- Active Directory Domain Services (AD DS)
- Domain: farhanlab.local
- Organizational Unit: IAM-Lab03/IT

## Tasks Completed
1. Opened Active Directory Users and Computers.
2. Created a Global Security Group named `it_support`.
3. Added John Carter and John Smith to the group.
4. Verified group membership using PowerShell.

## PowerShell Verification

```powershell
Get-ADGroupMember -Identity "it_support"
```

## Results
Successfully verified both group members:

- John Carter — john.carter
- John Smith — jsmith

## Skills Demonstrated
- Active Directory administration
- Security group management
- User-to-group assignment
- PowerShell verification
- Identity and Access Management fundamentals

## Screenshots
Include screenshots showing:
1. The security group in Active Directory.
2. The group membership configuration.
3. PowerShell output confirming both members.

## Conclusion
Successfully created and managed an Active Directory security group and verified group membership using PowerShell. This lab demonstrates foundational IAM administration skills used in enterprise environments.
