# PowerShell for Windows Management

## Overview

PowerShell is a powerful automation and configuration management framework from Microsoft. It's essential for managing Windows environments, particularly for Active Directory, Group Policy, and system administration tasks.

## Key Concepts

### Cmdlets

PowerShell commands follow a Verb-Noun naming convention:
- `Get-*` - Retrieve information
- `Set-*` - Modify configuration
- `New-*` - Create new objects
- `Remove-*` - Delete objects
- `Add-*` - Add to existing objects
- `Export-*` - Export data
- `Import-*` - Import data

### Pipeline

PowerShell's pipeline passes objects between commands:
```powershell
# Pipeline example
Get-ADUser -Filter * | Where-Object {$_.Enabled -eq $false} | Select-Object Name, LastLogonDate
```

### Modules

Load additional functionality:
```powershell
# Import Active Directory module
Import-Module ActiveDirectory

# List available modules
Get-Module -ListAvailable
```

## Active Directory Management

### User Management

#### Get User Information
```powershell
# Get single user
Get-ADUser -Identity "jsmith" -Properties *

# Get all users
Get-ADUser -Filter * -Properties DisplayName, EmailAddress, LastLogonDate

# Get users from specific OU
Get-ADUser -SearchBase "OU=Sales,DC=contoso,DC=com" -Filter *

# Find disabled accounts
Get-ADUser -Filter {Enabled -eq $false} -Properties LastLogonDate |
    Select-Object Name, SamAccountName, LastLogonDate
```

#### Create Users
```powershell
# Create new user
New-ADUser -Name "John Smith" `
    -GivenName "John" `
    -Surname "Smith" `
    -SamAccountName "jsmith" `
    -UserPrincipalName "jsmith@contoso.com" `
    -Path "OU=Users,DC=contoso,DC=com" `
    -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
    -Enabled $true `
    -ChangePasswordAtLogon $true

# Bulk create from CSV
Import-Csv "C:\users.csv" | ForEach-Object {
    New-ADUser -Name $_.Name `
        -SamAccountName $_.SamAccountName `
        -UserPrincipalName $_.UserPrincipalName `
        -Path $_.OU `
        -AccountPassword (ConvertTo-SecureString $_.Password -AsPlainText -Force) `
        -Enabled $true
}
```

#### Modify Users
```powershell
# Set single property
Set-ADUser -Identity "jsmith" -Department "IT"

# Set multiple properties
Set-ADUser -Identity "jsmith" -Department "IT" -Title "System Administrator" -Office "Building A"

# Disable user
Disable-ADAccount -Identity "jsmith"

# Enable user
Enable-ADAccount -Identity "jsmith"

# Reset password
Set-ADAccountPassword -Identity "jsmith" -Reset -NewPassword (ConvertTo-SecureString "NewP@ssw0rd!" -AsPlainText -Force)

# Force password change at next logon
Set-ADUser -Identity "jsmith" -ChangePasswordAtLogon $true
```

### Group Management

#### Get Group Information
```powershell
# Get group details
Get-ADGroup -Identity "Domain Admins" -Properties *

# Get all groups
Get-ADGroup -Filter *

# Get groups by type
Get-ADGroup -Filter {GroupCategory -eq "Security"}

# Get group membership
Get-ADGroupMember -Identity "Domain Admins"

# Get nested group membership
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

#### Manage Group Membership
```powershell
# Add user to group
Add-ADGroupMember -Identity "IT Staff" -Members "jsmith"

# Add multiple users to group
Add-ADGroupMember -Identity "IT Staff" -Members "jsmith", "bjones", "mwilliams"

# Remove user from group
Remove-ADGroupMember -Identity "IT Staff" -Members "jsmith" -Confirm:$false

# Get user's group memberships
Get-ADPrincipalGroupMembership -Identity "jsmith" | Select-Object Name
```

#### Create Groups
```powershell
# Create security group
New-ADGroup -Name "Database Admins" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Groups,DC=contoso,DC=com" `
    -Description "Database administrators"

# Create distribution group
New-ADGroup -Name "All Staff" `
    -GroupScope Universal `
    -GroupCategory Distribution `
    -Path "OU=Distribution Lists,DC=contoso,DC=com"
```

### Organizational Unit (OU) Management

```powershell
# Create OU
New-ADOrganizationalUnit -Name "Sales" -Path "DC=contoso,DC=com"

# Get OUs
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Move user to different OU
Move-ADObject -Identity "CN=John Smith,OU=Users,DC=contoso,DC=com" `
    -TargetPath "OU=Sales,DC=contoso,DC=com"
```

## Group Policy Management

### GPO Cmdlets

```powershell
# Import Group Policy module
Import-Module GroupPolicy

# Get all GPOs
Get-GPO -All

# Get specific GPO
Get-GPO -Name "Default Domain Policy"

# Create new GPO
New-GPO -Name "Security Policy" -Comment "Enhanced security settings"

# Link GPO to OU
New-GPLink -Name "Security Policy" -Target "OU=Workstations,DC=contoso,DC=com"

# Unlink GPO
Remove-GPLink -Name "Security Policy" -Target "OU=Workstations,DC=contoso,DC=com"

# Generate GPO report
Get-GPOReport -Name "Default Domain Policy" -ReportType Html -Path "C:\GPO-Report.html"

# Generate all GPOs report
Get-GPOReport -All -ReportType Html -Path "C:\All-GPOs-Report.html"
```

### GPResult

```powershell
# Get Group Policy results for local computer
gpresult /Scope Computer /v

# Get Group Policy results for user
gpresult /Scope User /v

# Generate HTML report
gpresult /H C:\gpresult.html

# Remote computer GPResult
Invoke-Command -ComputerName "PC001" -ScriptBlock { gpresult /H C:\gpresult.html }
```

## Computer Management

### Get Computer Information

```powershell
# Get AD computers
Get-ADComputer -Filter * -Properties OperatingSystem, LastLogonDate

# Get computers in specific OU
Get-ADComputer -SearchBase "OU=Workstations,DC=contoso,DC=com" -Filter *

# Get inactive computers
$DaysInactive = 90
$InactiveDate = (Get-Date).AddDays(-$DaysInactive)
Get-ADComputer -Filter {LastLogonDate -lt $InactiveDate} -Properties LastLogonDate |
    Select-Object Name, LastLogonDate
```

### Remote Management

```powershell
# Test connectivity
Test-Connection -ComputerName "PC001"

# Remote PowerShell session
Enter-PSSession -ComputerName "PC001"

# Run command on remote computer
Invoke-Command -ComputerName "PC001" -ScriptBlock { Get-Service }

# Run command on multiple computers
Invoke-Command -ComputerName "PC001", "PC002", "PC003" -ScriptBlock {
    Get-EventLog -LogName System -Newest 10
}

# Copy file to remote computer
Copy-Item -Path "C:\script.ps1" -Destination "\\PC001\C$\Scripts\" -Force
```

## Reporting and Auditing

### User Account Reports

```powershell
# Find users with passwords that never expire
Get-ADUser -Filter {PasswordNeverExpires -eq $true} -Properties PasswordNeverExpires |
    Select-Object Name, SamAccountName, PasswordNeverExpires |
    Export-Csv "C:\Reports\PasswordNeverExpires.csv" -NoTypeInformation

# Find users who haven't logged on in X days
$DaysInactive = 90
$InactiveDate = (Get-Date).AddDays(-$DaysInactive)
Get-ADUser -Filter {LastLogonDate -lt $InactiveDate} -Properties LastLogonDate |
    Select-Object Name, SamAccountName, LastLogonDate |
    Export-Csv "C:\Reports\InactiveUsers.csv" -NoTypeInformation

# Find users with expired passwords
Get-ADUser -Filter {Enabled -eq $true} -Properties PasswordLastSet, PasswordNeverExpires |
    Where-Object {$_.PasswordNeverExpires -eq $false} |
    Select-Object Name, SamAccountName, PasswordLastSet |
    Export-Csv "C:\Reports\PasswordStatus.csv" -NoTypeInformation
```

### Group Membership Audit

```powershell
# Export all members of privileged groups
$PrivilegedGroups = @(
    "Domain Admins",
    "Enterprise Admins",
    "Schema Admins",
    "Account Operators"
)

$Report = foreach ($Group in $PrivilegedGroups) {
    Get-ADGroupMember -Identity $Group -Recursive | Select-Object @{
        Name = "Group"
        Expression = {$Group}
    }, Name, SamAccountName, objectClass
}

$Report | Export-Csv "C:\Reports\PrivilegedGroupMembers.csv" -NoTypeInformation
```

### Security Audit

```powershell
# Find admin accounts
Get-ADUser -Filter * -Properties AdminCount |
    Where-Object {$_.AdminCount -gt 0} |
    Select-Object Name, SamAccountName, Enabled |
    Export-Csv "C:\Reports\AdminAccounts.csv" -NoTypeInformation

# Check for accounts with SPNs (potential Kerberoasting targets)
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName |
    Select-Object Name, SamAccountName, ServicePrincipalName |
    Export-Csv "C:\Reports\AccountsWithSPNs.csv" -NoTypeInformation
```

## Automation Scripts

### Daily User Cleanup

```powershell
# Script to disable inactive user accounts
$InactiveDays = 90
$InactiveDate = (Get-Date).AddDays(-$InactiveDays)
$TargetOU = "OU=Users,DC=contoso,DC=com"

# Get inactive enabled users
$InactiveUsers = Get-ADUser -Filter {
    Enabled -eq $true -and LastLogonDate -lt $InactiveDate
} -SearchBase $TargetOU -Properties LastLogonDate

foreach ($User in $InactiveUsers) {
    # Disable the account
    Disable-ADAccount -Identity $User
    
    # Add description noting when and why disabled
    Set-ADUser -Identity $User -Description "Disabled $(Get-Date -Format 'yyyy-MM-dd') - Inactive for $InactiveDays days"
    
    # Log action
    Write-Host "Disabled: $($User.SamAccountName) - Last logon: $($User.LastLogonDate)"
}
```

### Group Membership Sync

```powershell
# Sync group membership based on department
$Departments = @("IT", "Sales", "HR", "Finance")

foreach ($Dept in $Departments) {
    $GroupName = "$Dept Team"
    
    # Get users in department
    $DeptUsers = Get-ADUser -Filter {Department -eq $Dept} -Properties Department
    
    # Get current group members
    $GroupMembers = Get-ADGroupMember -Identity $GroupName
    
    # Add users not in group
    $UsersToAdd = $DeptUsers | Where-Object {$GroupMembers.SamAccountName -notcontains $_.SamAccountName}
    foreach ($User in $UsersToAdd) {
        Add-ADGroupMember -Identity $GroupName -Members $User
        Write-Host "Added $($User.SamAccountName) to $GroupName"
    }
    
    # Remove users no longer in department
    $UsersToRemove = $GroupMembers | Where-Object {$DeptUsers.SamAccountName -notcontains $_.SamAccountName}
    foreach ($User in $UsersToRemove) {
        Remove-ADGroupMember -Identity $GroupName -Members $User -Confirm:$false
        Write-Host "Removed $($User.SamAccountName) from $GroupName"
    }
}
```

## Error Handling

```powershell
# Try-Catch for error handling
try {
    Get-ADUser -Identity "nonexistent" -ErrorAction Stop
}
catch {
    Write-Host "Error: $($_.Exception.Message)" -ForegroundColor Red
}

# Test if object exists before operating
if (Get-ADUser -Filter {SamAccountName -eq "jsmith"}) {
    Set-ADUser -Identity "jsmith" -Department "IT"
}
else {
    Write-Host "User not found"
}
```

## Best Practices

1. **Use WhatIf and Confirm**
   ```powershell
   # Preview changes without executing
   Remove-ADUser -Identity "testuser" -WhatIf
   
   # Prompt for confirmation
   Remove-ADUser -Identity "testuser" -Confirm
   ```

2. **Use Splatting for Readability**
   ```powershell
   $UserParams = @{
       Name = "John Smith"
       SamAccountName = "jsmith"
       UserPrincipalName = "jsmith@contoso.com"
       Path = "OU=Users,DC=contoso,DC=com"
       Enabled = $true
   }
   New-ADUser @UserParams
   ```

3. **Log Actions**
   ```powershell
   # Log to file
   Start-Transcript -Path "C:\Logs\script-$(Get-Date -Format 'yyyyMMdd-HHmmss').log"
   # Your script here
   Stop-Transcript
   ```

4. **Use Progress Indicators**
   ```powershell
   $Users = Get-ADUser -Filter *
   $i = 0
   foreach ($User in $Users) {
       $i++
       Write-Progress -Activity "Processing Users" -Status "$i of $($Users.Count)" -PercentComplete (($i / $Users.Count) * 100)
       # Process user
   }
   ```

5. **Secure Credentials**
   ```powershell
   # Never store plain text passwords in scripts
   # Use secure string
   $SecurePassword = Read-Host "Enter password" -AsSecureString
   
   # Or use credential object
   $Credential = Get-Credential
   ```

## Resources

- PowerShell Gallery: https://www.powershellgallery.com/
- Microsoft PowerShell Documentation
- Active Directory PowerShell Module Reference
- PowerShell Community Resources

## Common Modules

- **ActiveDirectory** - AD management
- **GroupPolicy** - GPO management  
- **PSWindowsUpdate** - Windows Update management
- **ImportExcel** - Excel file manipulation
- **Posh-SSH** - SSH connectivity
