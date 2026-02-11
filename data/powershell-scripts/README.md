# PowerShell Scripts Directory

Store your PowerShell automation scripts for Windows management in this directory.

## Script Categories

- User management automation
- Group management automation
- GPO deployment and configuration
- Security auditing
- Reporting scripts
- Maintenance scripts

## Script Organization

Consider organizing scripts in subdirectories:
- `user-management/` - User account scripts
- `group-management/` - Group membership scripts
- `reporting/` - Audit and reporting scripts
- `gpo-management/` - Group Policy scripts
- `security/` - Security hardening and auditing
- `maintenance/` - Routine maintenance tasks

## Best Practices

1. **Include Comments**: Document what the script does
2. **Error Handling**: Use try-catch blocks
3. **Logging**: Log important actions
4. **Parameters**: Make scripts reusable with parameters
5. **Testing**: Test in non-production first
6. **Security**: Never hard-code passwords

## Example Script Template

```powershell
<#
.SYNOPSIS
    Brief description of what the script does

.DESCRIPTION
    Detailed description of the script's functionality

.PARAMETER ParameterName
    Description of parameter

.EXAMPLE
    .\Script-Name.ps1 -ParameterName "Value"
    Description of what this example does

.NOTES
    Author: Your Name
    Date: 2024-02-11
    Version: 1.0
#>

[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [string]$ParameterName
)

# Start logging
Start-Transcript -Path "C:\Logs\Script-$(Get-Date -Format 'yyyyMMdd-HHmmss').log"

try {
    # Your script logic here
    Write-Host "Starting script..." -ForegroundColor Green
    
    # Main functionality
    
    Write-Host "Script completed successfully" -ForegroundColor Green
}
catch {
    Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host $_.ScriptStackTrace
}
finally {
    # Cleanup
    Stop-Transcript
}
```

## What to Ask Claude

Example requests for PowerShell scripting help:
- "Create a script to audit inactive user accounts"
- "Write a script to bulk create users from a CSV file"
- "Generate a report of all group memberships"
- "Create a maintenance script to clean up old computer accounts"
- "Write a script to backup GPO settings"
- "Help me automate password expiration notifications"
