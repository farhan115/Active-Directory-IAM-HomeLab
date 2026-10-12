# Lab 07 — Active Directory Password Reset & Account Lockout Management

## Objective
Demonstrate password reset administration, account lockout troubleshooting, account unlocking, and security policy verification using Active Directory and PowerShell.

## Lab Environment
- Windows Server 2025
- VMware Workstation
- Active Directory Domain Services (AD DS)
- Domain: farhanlab.local
- Domain Controller: Win2K25-DC01
- Test Account: John Smith (`jsmith`)

## Part 1 — Password Reset Verification

Configured John Smith's account to require a password change at the next logon.

```powershell
Get-ADUser -Identity "jsmith" -Properties PasswordLastSet,PasswordExpired,Enabled,LockedOut |
Select-Object Name,Enabled,LockedOut,PasswordLastSet,PasswordExpired
```

**Results:**
- Enabled: True
- LockedOut: False
- PasswordExpired: True
- pwdLastSet: 0

## Part 2 — Configure Account Lockout Policy

Using Group Policy Management, configured:

| Setting | Value |
|---|---|
| Account lockout threshold | 5 failed attempts |
| Account lockout duration | 10 minutes |
| Reset account lockout counter after | 10 minutes |

Applied and verified the policy:

```powershell
gpupdate /force

Get-ADDefaultDomainPasswordPolicy |
Format-List LockoutThreshold,LockoutDuration,LockoutObservationWindow
```

## Part 3 — Simulate Account Lockout

Performed controlled failed authentication attempts against the test account.

```powershell
Add-Type -AssemblyName System.DirectoryServices.AccountManagement

$context = New-Object System.DirectoryServices.AccountManagement.PrincipalContext(
    [System.DirectoryServices.AccountManagement.ContextType]::Domain,
    "farhanlab.local"
)

$context.ValidateCredentials("jsmith", "IncorrectLabPassword123!")
```

Verified the account status:

```powershell
Get-ADUser -Identity "jsmith" -Properties LockedOut,badPwdCount |
Select-Object Name,LockedOut,badPwdCount
```

**Results:**
- Failed password count: 5
- LockedOut: True

## Part 4 — Unlock Account

Unlocked John Smith's account using PowerShell:

```powershell
Unlock-ADAccount -Identity "jsmith"
```

Verified:

```powershell
Get-ADUser -Identity "jsmith" -Properties Enabled,LockedOut |
Select-Object Name,Enabled,LockedOut
```

**Results:**
- Enabled: True
- LockedOut: False

## Part 5 — Restore Original Security Policy

Restored the original account lockout threshold to 0 in Default Domain Policy.

```powershell
gpupdate /force

Get-ADDefaultDomainPasswordPolicy |
Format-List LockoutThreshold,LockoutDuration,LockoutObservationWindow
```

**Final verification:** LockoutThreshold = 0

## Troubleshooting Experience

- Investigated a domain lockout policy that initially did not reflect the configured settings.
- Applied Group Policy updates and verified the effective domain configuration.
- Encountered and investigated an LDAP server unavailable error during authentication testing.
- Verified failed authentication counts and successful account recovery.

## Skills Demonstrated

- Active Directory administration
- Password reset and account recovery
- Account lockout troubleshooting
- Group Policy Management
- PowerShell administration
- Authentication troubleshooting
- IAM security operations

## Conclusion

Successfully demonstrated password reset configuration, account lockout simulation, account unlocking, and restoration of the original domain security policy in a controlled Windows Server Active Directory lab.
