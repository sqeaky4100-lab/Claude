# User Exports Directory

Store Active Directory user account exports in this directory for analysis.

## Supported File Types

- `.csv` - CSV exports (preferred)
- `.json` - JSON exports
- `.txt` - Plain text exports
- `.xml` - XML exports

## How to Generate User Exports

### Basic User Export

```powershell
# Export all users with common properties
Get-ADUser -Filter * -Properties DisplayName, EmailAddress, Department, Title, LastLogonDate, PasswordLastSet, Enabled |
    Select-Object Name, SamAccountName, DisplayName, EmailAddress, Department, Title, LastLogonDate, PasswordLastSet, Enabled |
    Export-Csv "users-all-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Specific User Groups

```powershell
# Export disabled users
Get-ADUser -Filter {Enabled -eq $false} -Properties LastLogonDate |
    Select-Object Name, SamAccountName, LastLogonDate |
    Export-Csv "users-disabled-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export inactive users (no logon in 90 days)
$InactiveDate = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $InactiveDate} -Properties LastLogonDate |
    Select-Object Name, SamAccountName, LastLogonDate, Enabled |
    Export-Csv "users-inactive-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export privileged users
Get-ADUser -Filter * -Properties AdminCount, MemberOf |
    Where-Object {$_.AdminCount -gt 0} |
    Select-Object Name, SamAccountName, Enabled |
    Export-Csv "users-privileged-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Users from Specific OU

```powershell
# Export users from specific OU
Get-ADUser -SearchBase "OU=Sales,DC=contoso,DC=com" -Filter * -Properties Department, Manager |
    Select-Object Name, SamAccountName, Department, Manager, Enabled |
    Export-Csv "users-sales-ou-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Comprehensive Export

```powershell
# Export users with extensive properties
Get-ADUser -Filter * -Properties * |
    Select-Object Name, SamAccountName, DisplayName, EmailAddress, Department, Title, 
                  Manager, OfficePhone, MobilePhone, Office, Company, 
                  Created, Modified, LastLogonDate, PasswordLastSet, PasswordNeverExpires,
                  PasswordExpired, LockedOut, Enabled, AdminCount |
    Export-Csv "users-comprehensive-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

## File Naming Conventions

Use descriptive names with dates:
- `users-all-2024-02-11.csv`
- `users-disabled-2024-02-11.csv`
- `users-sales-department-2024-02-11.csv`
- `users-privileged-2024-02-11.csv`

## What to Ask Claude

Example questions for user account analysis:
- "Identify inactive accounts that should be disabled"
- "Which users have passwords that never expire?"
- "Are there any naming convention issues with these accounts?"
- "Identify accounts with potential security risks"
- "Which users haven't logged in for over 90 days?"
- "Are there duplicate or test accounts?"
