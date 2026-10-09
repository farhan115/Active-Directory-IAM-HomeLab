# Lab 04 – IAM User Provisioning Automation with PowerShell

## Project Overview

This lab demonstrates how to automate a simulated Identity and Access Management (IAM) user provisioning process using Windows PowerShell and a CSV file.

The goal is to understand how IAM administrators automate employee onboarding, generate usernames, validate identity information, and process multiple user records efficiently.

The lab was completed in a Windows Server 2025 virtual machine.

**Lab Status:** Successfully Completed

**Lab Type:** IAM Automation / Identity Lifecycle Management

**Technology:** Windows PowerShell, CSV, Windows Server 2025

---

## 1. Lab Objectives

The objectives of this lab were to:

- Create a CSV file containing employee information.
- Import employee records into PowerShell.
- Automate processing of multiple employee identities.
- Generate usernames using first and last names.
- Validate employee information before processing.
- Simulate successful account provisioning.
- Verify the PowerShell script executes correctly.
- Understand how automation supports IAM operations.

---

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Operating System | Windows Server 2025 |
| Virtual Machine | Win2K25-DC01 |
| Scripting Language | Windows PowerShell |
| Data Source | CSV |
| Script | Lab04-UserProvisioning.ps1 |
| CSV File | users.csv |
| Working Directory | C:\IAM-Labs\Lab04 |
| Version Control | GitHub |

The lab was performed in a Windows Server virtual machine with PowerShell.

No actual Active Directory or Microsoft Entra ID accounts were created during this lab.

---

## 3. Business Scenario

An organization hires three employees for different departments:

- John Smith – IT
- Sarah Johnson – HR
- Michael Brown – Finance

The IAM team must process their identity information as part of the employee onboarding workflow.

Instead of processing each employee manually, the administrator develops a PowerShell script that reads employee records from a CSV file and automatically generates usernames.

This exercise simulates the initial stages of enterprise identity provisioning.

---

## 4. Step 1 – Create the Working Directory

Opened PowerShell and executed:

```powershell
New-Item -Path "C:\IAM-Labs\Lab04" -ItemType Directory -Force
```

Navigated to the directory:

```powershell
Set-Location "C:\IAM-Labs\Lab04"
```

**Result:** Successfully created the Lab04 working directory.

---

## 5. Step 2 – Create the Employee CSV File

Created a CSV file named `users.csv` containing three employee records.

### CSV Data

```csv
FirstName,LastName,Department
John,Smith,IT
Sarah,Johnson,HR
Michael,Brown,Finance
```

### PowerShell Command

```powershell
@"
FirstName,LastName,Department
John,Smith,IT
Sarah,Johnson,HR
Michael,Brown,Finance
"@ | Set-Content -Path ".\users.csv"
```

### Verify CSV Data

```powershell
Import-Csv .\users.csv
```

The CSV file serves as the input source for the provisioning automation.

---

## 6. Step 3 – Create the PowerShell Provisioning Script

Created the script:

`Lab04-UserProvisioning.ps1`

### PowerShell Script

```powershell
# Lab 04 - IAM User Provisioning Automation
# Purpose: Simulate identity provisioning from CSV data

$users = Import-Csv "$PSScriptRoot\users.csv"

foreach ($user in $users) {

    $firstName = $user.FirstName.Trim()
    $lastName = $user.LastName.Trim()
    $department = $user.Department.Trim()

    # Validate required employee information
    if ([string]::IsNullOrWhiteSpace($firstName) -or
        [string]::IsNullOrWhiteSpace($lastName) -or
        [string]::IsNullOrWhiteSpace($department)) {

        Write-Warning "Skipping incomplete user record"
        continue
    }

    # Generate username
    $username = (
        $firstName.Substring(0,1) + $lastName
    ).ToLower()

    # Simulate identity provisioning
    Write-Output "Provisioning user: $username"
    Write-Output "Department: $department"
    Write-Output "Status: Simulated account created"
    Write-Output "-----------------------------"
}
```

### Script Explanation

**Import-Csv**

Reads employee information from the CSV file and converts each row into a PowerShell object.

**foreach**

Processes each employee record individually.

**Trim()**

Removes unnecessary whitespace from employee information.

**IsNullOrWhiteSpace()**

Checks whether required identity information is missing or empty.

**Substring()**

Extracts the first character of an employee's first name.

**ToLower()**

Converts generated usernames into lowercase letters.

**Write-Output**

Displays the simulated provisioning results.

---

## 7. Step 4 – Execute the PowerShell Script

Initially, the PowerShell script file was empty, so executing it produced no output.

After identifying the issue, the script was opened in Notepad and the PowerShell code was saved correctly.

### Execution Command

```powershell
.\Lab04-UserProvisioning.ps1
```

### Successful Execution Output

```text
Provisioning user: jsmith
Department: IT
Status: Simulated account created
-----------------------------
Provisioning user: sjohnson
Department: HR
Status: Simulated account created
-----------------------------
Provisioning user: mbrown
Department: Finance
Status: Simulated account created
-----------------------------
```

**Result:** The script successfully processed all three employee records.

---

## 8. Lab Results

| Employee | Username | Department | Result |
|---|---|---|---|
| John Smith | jsmith | IT | Successful |
| Sarah Johnson | sjohnson | HR | Successful |
| Michael Brown | mbrown | Finance | Successful |

### Validation

- Three employee records were processed.
- Usernames were generated automatically.
- Employee departments were displayed correctly.
- No PowerShell execution errors appeared in the final run.
- The saved PowerShell script executed successfully.

**Overall Result: PASS**

The results represent simulated account provisioning, not the creation of actual directory accounts.

---

## 9. Troubleshooting

### Issue: PowerShell Script Produced No Output

After executing:

```powershell
.\Lab04-UserProvisioning.ps1
```

No output appeared.

### Investigation

Used the following command:

```powershell
Get-ChildItem C:\IAM-Labs\Lab04
```

The output showed:

```text
Length Name
------ ----
0      Lab04-UserProvisioning.ps1
84     users.csv
```

The PowerShell script file contained zero bytes, meaning the code had not been saved to the file.

### Resolution

Opened the script in Notepad:

```powershell
notepad C:\IAM-Labs\Lab04\Lab04-UserProvisioning.ps1
```

Pasted the PowerShell code and saved the file.

Executed the script again:

```powershell
.\Lab04-UserProvisioning.ps1
```

The script successfully processed all three users.

### Lesson Learned

Always verify that scripts are saved correctly before executing them.

Useful troubleshooting commands include:

```powershell
Get-ChildItem
Get-Content .\Lab04-UserProvisioning.ps1
Get-Content .\users.csv
```

---

## 10. IAM Security Considerations

In a production IAM environment, automated provisioning must follow organizational security policies.

Important controls include:

**Identity Validation:** Verify employee information before creating accounts.

**Least Privilege:** Grant only the access required for an employee's job responsibilities.

**Duplicate Account Prevention:** Check whether an identity or username already exists.

**Approval Workflows:** Require appropriate authorization before provisioning access.

**Audit Logging:** Maintain records of identity creation, modification, and deletion activities.

**Credential Security:** Never hardcode passwords or sensitive credentials in automation scripts.

**Access Reviews:** Periodically review whether employee permissions remain appropriate.

These controls help organizations reduce identity-related security risks.

---

## 11. Real-World IAM Applications

The concepts practiced in this lab apply to several enterprise IAM technologies.

### Microsoft Active Directory

PowerShell can automate user account creation using commands such as:

```powershell
New-ADUser
```

### Microsoft Entra ID

Microsoft Graph PowerShell can automate cloud identity provisioning and account management.

### SailPoint Identity Security Cloud

SailPoint supports identity lifecycle management, access governance, and provisioning workflows through configured integrations and policies.

### CyberArk

CyberArk solutions support privileged identity security and access management.

The CSV processing and scripting concepts practiced in this lab provide a foundation for working with these technologies.

---

## 12. Skills Demonstrated

**IAM Skills**
- Identity lifecycle management
- User provisioning concepts
- Employee onboarding workflows
- Identity data validation
- Access management fundamentals

**Technical Skills**
- Windows PowerShell
- CSV file processing
- PowerShell loops
- String manipulation
- Basic scripting
- Script execution
- Troubleshooting
- Windows Server administration

---

## 13. Project Files

```text
Lab-04-User-Provisioning/
|
|-- README.md
|-- users.csv
|-- Lab04-UserProvisioning.ps1
|-- execution-screenshot.png
```

The execution screenshot provides evidence of the successful PowerShell automation run.

---

## 14. Key Takeaways

Through this lab, I learned how to:

1. Organize an IAM automation project.
2. Create and import CSV employee data.
3. Write a PowerShell script to process multiple identities.
4. Generate standardized usernames.
5. Validate required employee attributes.
6. Troubleshoot an empty PowerShell script.
7. Execute and verify a saved automation script.
8. Document technical work for a GitHub portfolio.

This project strengthened my understanding of IAM automation and employee onboarding processes.

---

## 15. Future Improvements

Potential improvements include:

- Create actual Active Directory accounts using `New-ADUser`.
- Implement duplicate username detection.
- Add error handling with `try/catch`.
- Export provisioning results to an audit log.
- Assign security groups based on employee departments.
- Integrate with Microsoft Entra ID using Microsoft Graph.
- Automate user deprovisioning.
- Explore identity lifecycle workflows in SailPoint.

---

## Conclusion

Successfully completed Lab 04 – IAM User Provisioning Automation using PowerShell on Windows Server 2025.

The script imported employee information from a CSV file, generated usernames, validated identity attributes, and simulated provisioning for three employees.

This lab demonstrates foundational PowerShell automation skills relevant to entry-level IAM Analyst, Identity Administrator, and IT Support positions.

**Lab Status: Completed Successfully**
