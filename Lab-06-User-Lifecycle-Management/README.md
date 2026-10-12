# Lab 06 — Active Directory User Lifecycle Management

## Objective
Simulate the Joiner–Mover–Leaver (JML) identity lifecycle using Active Directory and PowerShell.

## Lab Environment
- Windows Server 2025
- VMware Workstation
- Active Directory Domain Services (AD DS)
- Domain: farhanlab.local
- Organizational Units: IAM-Lab03/IT and IAM-Lab03/HR

## Part 1 — Employee Onboarding (Joiner)
- Created Sarah Johnson (`sarah.johnson`) in the IT OU.
- Added Sarah to the `it_support` security group.
- Verified group membership through Active Directory Users and Computers.

## Part 2 — Department Transfer (Mover)
- Created the `Hr_support` security group.
- Added Sarah to the HR group.
- Removed Sarah from `it_support`.
- Moved Sarah's account from the IT OU to the HR OU.

### PowerShell Commands

```powershell
Remove-ADGroupMember -Identity "it_support" -Members "sarah.johnson"

Get-ADUser -Identity "sarah.johnson" |
Move-ADObject -TargetPath "OU=HR,OU=IAM-Lab03,DC=farhanlab,DC=local"

Get-ADPrincipalGroupMembership "sarah.johnson" |
Select-Object Name
```

## Part 3 — Employee Offboarding (Leaver)
- Disabled Sarah's Active Directory account.
- Removed her HR security group membership.
- Verified that the account was disabled and only default group membership remained.

### PowerShell Commands

```powershell
Disable-ADAccount -Identity "sarah.johnson"

Get-ADUser -Identity "sarah.johnson" -Properties Enabled |
Select-Object Name,SamAccountName,Enabled

Remove-ADGroupMember -Identity "Hr_support" -Members "sarah.johnson"

Get-ADPrincipalGroupMembership "sarah.johnson" |
Select-Object Name
```

## Results
- Account status: Enabled = False
- HR security group access: Removed
- Remaining group membership: Domain Users
- Employee account retained in Active Directory for administrative purposes.

## Skills Demonstrated
- Identity lifecycle management (JML)
- Active Directory user provisioning
- Department transfers
- Security group management
- Account deprovisioning
- PowerShell administration
- IAM access verification

## Conclusion
Successfully simulated the full Joiner–Mover–Leaver lifecycle in Active Directory. Demonstrated the ability to provision, transfer, disable, and update access for employee accounts using graphical tools and PowerShell.
