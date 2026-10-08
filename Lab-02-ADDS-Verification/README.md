# Lab 02 — Active Directory Domain Services Verification

## Objective
Verify that the Windows Server 2025 domain controller is functioning and that essential Active Directory services are running.

## Lab Environment
- Hypervisor: VMware Workstation
- Operating System: Windows Server 2025
- Domain Controller: Win2K25-DC01
- Domain: farhanlab.local
- Domain Functional Level: Windows2025Domain

## Verification 1 — Active Directory Domain

Executed the following PowerShell command:

`Get-ADDomain`

**Result:** Successfully retrieved the Active Directory domain configuration.

## Verification 2 — Critical Services

Executed:

`Get-Service NTDS,DNS,Netlogon | Select-Object Name,Status,StartType`

**Results:**

| Service | Status | Startup Type |
|---|---|---|
| DNS | Running | Automatic |
| Netlogon | Running | Automatic |
| NTDS | Running | Automatic |

## Skills Demonstrated
- Active Directory administration
- Windows Server management
- PowerShell administration
- Domain controller service verification
- IAM infrastructure documentation

## Next Steps
- Investigate Server Manager alerts
- Validate DNS using DCDIAG
- Create Organizational Units
- Configure users and security groups
