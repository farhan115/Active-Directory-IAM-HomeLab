# Active Directory Password Reset Delegation Lab

**Author:** [Farhan Muntasir](https://github.com/farhan115)  
**Environment:** Windows Server 2025 lab VM, Active Directory Domain Services, ADUC, Windows PowerShell  
**Status:** Completed — authorized reset succeeded and out-of-scope reset was denied

## Overview

Implemented OU-scoped password-reset delegation for a help desk security group in a personal Active Directory lab. Created separate locations for help desk identities, managed users, and an out-of-scope comparison user. Used explicit help desk credentials in PowerShell to verify both an allowed reset and an access-denied result.

The project demonstrates a practical least-privilege administration pattern without granting the help desk user Domain Admin membership. Evidence established the tested password-reset boundary; it did not establish that every other directory permission was audited.

## Business scenario

A help desk team needs to reset employee passwords without receiving domain-wide administrative privileges. The requirement is to delegate that task within a designated OU and verify that it does not extend to a comparison user in a sibling OU.

## Lab inventory

| Object | Value |
|---|---|
| Domain | `farhanlab.local` |
| Domain controller | `Win2K25-DC01` |
| Parent OU | `LAB-Delegation` |
| Help desk OU | `Helpdesk` |
| Delegated OU | `Managed-Users` |
| Comparison OU | `Outside-Scope` |
| Help desk account | `lab.helpdesk1` |
| Delegation group | `GG-LAB-PasswordReset` |
| Group scope / type | Global / Security |
| Managed account | `lab.employee1` |
| Outside-scope account | `lab.outside1` |
| Delegated wizard task | Reset user passwords and force password change at next logon |

## OU structure

| OU path beneath farhanlab.local | Contents |
|---|---|
| `LAB-Delegation/Helpdesk` | Help desk user and delegation group |
| `LAB-Delegation/Managed-Users` | `lab.employee1` |
| `LAB-Delegation/Outside-Scope` | `lab.outside1` |

## Prerequisites

- A working Windows Server AD DS lab and administrator access for setup.
- Active Directory Users and Computers and the ActiveDirectory PowerShell module.
- Newly created ordinary test accounts and passwords satisfying the domain policy.
- Connectivity to the named domain controller.

These commands change test account passwords. Run them only against the lab identities created for this exercise. Keep passwords out of scripts, screenshots, and Git commits.

## 1. Create the OU structure

On the domain controller, open Run and enter:

```text
dsa.msc
```

Right-click `farhanlab.local` → New → Organizational Unit. Create `LAB-Delegation`, then create its three child OUs: `Helpdesk`, `Managed-Users`, and `Outside-Scope`. Keep protection from accidental deletion enabled when creating the OUs.

## 2. Create test users

Right-click the appropriate OU → New → User.

| OU | Display name | Logon name |
|---|---|---|
| Helpdesk | Lab Helpdesk 1 | `lab.helpdesk1` |
| Managed-Users | Lab Employee 1 | `lab.employee1` |
| Outside-Scope | Lab Outside 1 | `lab.outside1` |

Use lab-only passwords. For this short exercise, clear User must change password at next logon. Leave User cannot change password, Password never expires, and Account is disabled unchecked.

The initial creation dialog had Password never expires and Account is disabled selected. The corrective instructions were to clear both before finishing. Successful later use of the help desk credentials supports that the account could perform the tested operation.

## 3. Create and populate the help desk group

In `Helpdesk`, create a Global Security group named `GG-LAB-PasswordReset`.

Open Properties → Members → Add. Add `lab.helpdesk1`, using Check Names to resolve it.

Verify from administrator PowerShell:

```powershell
Import-Module ActiveDirectory
Get-ADGroupMember GG-LAB-PasswordReset |
    Select-Object Name, SamAccountName
```

Observed member: `lab.helpdesk1`.

Verify the managed user's location:

```powershell
Get-ADUser lab.employee1 |
    Select-Object Name, DistinguishedName
```

The returned distinguished name placed Lab Employee 1 under `OU=Managed-Users,OU=LAB-Delegation`.

## 4. Delegate the password-reset task

1. Right-click **Managed-Users**, not the domain or `LAB-Delegation` parent.
2. Choose Delegate Control → Next → Add.
3. Add `GG-LAB-PasswordReset` and resolve it with Check Names.
4. Choose Delegate the following common tasks.
5. Select **Reset user passwords and force password change at next logon** only.
6. Review the summary and click Finish.

The verified completion summary showed the scope `farhanlab.local/LAB-Delegation/Managed-Users`, the correct group, and the password-reset task.

The wizard task includes the ability to force a password change. The behavioral tests below exercised password reset only; forcing a password change was not separately tested.

## 5. Prepare explicit help desk credentials

Keep the administrator session open for setup and inspection. The reset cmdlets below explicitly use the delegated account through `-Credential`.

Use terminal-based secure input:

```powershell
Import-Module ActiveDirectory

$helpdeskPassword = Read-Host "Enter lab.helpdesk1's password" -AsSecureString
$helpdeskCred = [System.Management.Automation.PSCredential]::new(
    "lab.helpdesk1@farhanlab.local",
    $helpdeskPassword
)

$helpdeskCred.UserName
$newPassword = Read-Host "Enter a new password for the test account" -AsSecureString
```

Expected username output:

```text
lab.helpdesk1@farhanlab.local
```

`$helpdeskPassword` authenticates the help desk account. `$newPassword` is the replacement password for the target test account. These serve different purposes.

Displaying UserName confirms the credential object is populated; it does not independently validate the password or account permissions.

## 6. Positive test — reset the managed user

```powershell
Set-ADAccountPassword -Identity "lab.employee1" -Reset -NewPassword $newPassword -Credential $helpdeskCred -Server "Win2K25-DC01" -ErrorAction Stop
```

**Observed result:** the prompt returned without an error. The cmdlet normally produces no output on success.

## 7. Negative test — attempt an outside-scope reset

```powershell
Set-ADAccountPassword -Identity "lab.outside1" -Reset -NewPassword $newPassword -Credential $helpdeskCred -Server "Win2K25-DC01" -ErrorAction Stop
```

**Observed result:** `Access is denied`, with UnauthorizedAccessException / PermissionDenied details.

This was the expected result. The delegated help desk credentials could reset the managed user but could not reset the comparison user outside the delegated OU.

## Validation results

| Check | Observed result | Status |
|---|---|---|
| Help desk group membership | `lab.helpdesk1` listed | Passed |
| Managed user location | Under Managed-Users OU | Passed |
| Wizard summary | Correct OU, group, and task | Verified |
| Credential object | Expected UserName displayed | Populated |
| Reset managed user | No error | Passed |
| Reset outside-scope user | Access is denied | Passed |

Successful use of explicit credentials is the basis for the behavioral test. The elevated window title alone does not identify the credentials the reset cmdlet used.

## Troubleshooting encountered

### Empty Credential argument

The first attempts used Get-Credential, but `$helpdeskCred` remained empty. Referencing `$helpdeskCred.UserName` returned no value, and Set-ADAccountPassword failed parameter validation before a reset was attempted.

The exact reason the dialog did not yield a credential was not established. The working alternative used Read-Host -AsSecureString and constructed a PSCredential object directly.

### Access denied for the managed account

After credentials were populated, the managed-user reset initially returned Access is denied. Checked the employee's OU location and the help desk account's group membership; both were correct.

Revisited the Delegation of Control Wizard, selected the password-reset task for the correct group on Managed-Users, verified the final summary, and finished. The next managed-user reset succeeded, while the outside-scope reset remained denied.

The evidence shows success after completing this delegation. It does not establish the exact defect in the earlier delegation attempt, because its ACL was not captured.

### If denial persists in another run

Inspect the actual OU and user ACLs, permission inheritance, explicit deny entries, group membership, and whether the target is or was a protected administrative account. Verify the same domain controller is being used and account for replication if the lab has multiple DCs. Do not grant broad administrator membership simply to make the test pass.

These are diagnostic follow-ups, not additional tasks completed in this lab.

## Interview questions and answers

### 1. What did you build?
I delegated password-reset administration to a help desk security group for one OU and used PowerShell with explicit help desk credentials to validate an allowed reset and an outside-scope denial.

### 2. Why delegate instead of granting Domain Admin?
The help desk needs a specific task over a limited set of users. Domain Admin would give far broader authority than that requirement.

### 3. What is an OU?
An organizational unit is an AD container used to organize directory objects and scope administrative delegation and Group Policy links. This lab used sibling OUs to separate the intended scope from the comparison account.

### 4. What is the difference between the OU and the security group?
The OU contains the objects receiving delegated permissions through its hierarchy. The security group identifies the principals receiving those permissions. Group membership does not move a user into the OU.

### 5. Why use a security group for delegation?
Permissions can be managed through group membership rather than creating separate ACL entries for each technician. The group must be security-enabled for authorization.

### 6. Why was Global scope appropriate here?
The help desk account and resources were in one domain, so a Global Security group was suitable for this small lab. Larger environments may use a layered group design for resource permissions.

### 7. What does the Delegation of Control Wizard do?
It adds access control entries to support the selected tasks at the selected directory scope. It does not need to make the technician a domain administrator.

### 8. Why delegate on Managed-Users rather than the parent OU?
Delegating on the parent could extend permissions to other descendant OUs. The intended requirement was limited to Managed-Users.

### 9. What is the difference between changing and resetting a password?
Changing a password normally uses knowledge of the existing password. An authorized administrator or delegate can reset another user's password without knowing the previous one.

### 10. Why use -Credential in an administrator session?
It explicitly submits the operation using the help desk account rather than relying on the administrator session identity. Without it, a successful reset could simply demonstrate administrator authority.

### 11. What is PSCredential?
It is a PowerShell object containing a username and SecureString password, accepted by commands that support alternate credentials.

### 12. Why use Read-Host -AsSecureString?
It collects the password without putting its literal value in the command text. It avoids hard-coded passwords but does not remove the need to protect the session and computer.

### 13. What caused the Credential null-or-empty error?
The credential variable was empty. Parameter validation failed before the reset could execute. Constructing a populated PSCredential resolved that input problem.

### 14. Did displaying the username prove authentication succeeded?
No. It proved the credential object contained a username. The later successful directory operation provided evidence that those credentials worked for that operation.

### 15. How did you verify the managed user's location?
I queried Get-ADUser and inspected DistinguishedName, which showed the account beneath Managed-Users and LAB-Delegation.

### 16. How did you verify help desk membership?
I inspected the group's Members tab and used Get-ADGroupMember to display Name and SamAccountName.

### 17. How did you interpret no output from Set-ADAccountPassword?
The cmdlet normally returns no output on success. With ErrorAction Stop and no error, the managed-user reset completed successfully. I did not perform a separate sign-in with the replacement password.

### 18. Why was Access is denied a successful test result?
For the outside-scope account, denial was the expected boundary. It showed those credentials did not have password-reset authority over that comparison target.

### 19. Why use -Server?
It directs the command to the specified domain controller, helping keep the test endpoint consistent. It does not grant extra permissions.

### 20. What does -ErrorAction Stop do?
It makes nonterminating errors terminating for the command, which supports explicit error handling. It does not alter AD authorization.

### 21. What would you inspect if a correctly located user still cannot be reset?
The OU delegation entries, inherited user permissions, explicit denies, effective group memberships, protected-account status, and DC consistency. Those checks would follow rather than assuming the account needs broader privileges.

### 22. Did you verify forcing a password change at next logon?
No. The selected wizard task included that capability, but the behavioral tests executed password reset only.

### 23. What did the initial access-denied issue teach you?
Valid credentials, correct group membership, and correct OU placement are necessary checks, but they do not by themselves prove the intended ACL has been assigned. The delegation must also be completed and verified.

### 24. What are the project's limits?
This was one personal lab domain, one delegated group, and two password-reset scenarios. It did not audit every permission, test all users or nested OUs, collect security audit events, or implement a production help desk workflow.

### 25. How would you explain the value to a manager?
The help desk can perform a defined support task within an approved scope while a comparison account remains outside that authority. This reduces reliance on broad administrative access and makes the boundary testable.

## Resume bullet

- Configured Active Directory password-reset delegation for **1 help desk security group** on a dedicated OU in a Windows Server lab; validated **2 permission scenarios** with PowerShell, confirming an authorized reset and denial outside the delegated scope.

## GitHub details

**Repository name:** `active-directory-password-reset-delegation`

**Description:** Windows Server Active Directory lab demonstrating OU-scoped password-reset delegation, security group membership, explicit PowerShell credentials, and positive/negative permission testing.

**Suggested topics:** `active-directory`, `windows-server`, `powershell`, `iam`, `least-privilege`, `helpdesk`, `delegation`

This README contains no embedded images and needs no screenshots folder.

### Upload using GitHub

1. Create a repository using the suggested name, or open an existing AD lab repository.
2. Choose Add file → Upload files.
3. Upload this file and name it README.md in the repository or project folder.
4. Commit with: `Add AD password reset delegation and scope validation lab`.

## Cleanup

Close the test PowerShell session or remove its credential variables:

```powershell
Remove-Variable helpdeskPassword, helpdeskCred, newPassword -ErrorAction SilentlyContinue
```

This does not undo the delegation or password reset. If retiring the lab, review and remove only the explicit delegation entries created for GG-LAB-PasswordReset on Managed-Users using Advanced Security settings. Do not disable inheritance or remove unrelated permissions. Remove test objects only after checking for dependencies; protected OUs require deliberate removal of accidental-deletion protection before deletion.
