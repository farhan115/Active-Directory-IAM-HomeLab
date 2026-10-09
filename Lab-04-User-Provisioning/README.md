# Lab 04 – IAM User Provisioning Automation Using PowerShell

## Objective
Automate a simulated IAM user provisioning workflow using PowerShell and CSV data.

## Environment
- Windows Server 2025
- Windows PowerShell
- CSV file
- Windows Server virtual machine

## Lab Activities
1. Created a CSV file containing three employee records.
2. Imported employee information using `Import-Csv`.
3. Processed user records using a PowerShell `foreach` loop.
4. Generated usernames automatically.
5. Validated required employee fields.
6. Executed the saved PowerShell script.
7. Verified the simulated provisioning results.

## Results

| Username | Department | Status |
|---|---|---|
| jsmith | IT | Simulation successful |
| sjohnson | HR | Simulation successful |
| mbrown | Finance | Simulation successful |

## PowerShell Concepts
- `Import-Csv`
- `foreach`
- `Trim()`
- `Substring()`
- `ToLower()`
- `Write-Output`
- Input validation

## Security Considerations
In a production environment, user provisioning should include approval workflows, duplicate account checks, least-privilege access, secure credential handling, and audit logging.

## Important Note
This lab simulates account provisioning. It does not create actual Active Directory or Microsoft Entra ID accounts.

## Outcome
Successfully executed the PowerShell automation script and processed three employee records.

## Evidence
PowerShell execution screenshot demonstrating successful processing of all three users.
