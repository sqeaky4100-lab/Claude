# Group Exports Directory

Store Active Directory group and group membership exports in this directory for analysis.

## Supported File Types

- `.csv` - CSV exports (preferred)
- `.json` - JSON exports
- `.txt` - Plain text exports
- `.xml` - XML exports

## How to Generate Group Exports

### All Groups Export

```powershell
# Export all groups with properties
Get-ADGroup -Filter * -Properties Description, ManagedBy, GroupScope, GroupCategory, MemberCount |
    Select-Object Name, SamAccountName, Description, ManagedBy, GroupScope, GroupCategory |
    Export-Csv "groups-all-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Security vs Distribution Groups

```powershell
# Export security groups only
Get-ADGroup -Filter {GroupCategory -eq "Security"} -Properties Description |
    Select-Object Name, SamAccountName, Description, GroupScope |
    Export-Csv "groups-security-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export distribution groups
Get-ADGroup -Filter {GroupCategory -eq "Distribution"} -Properties Description |
    Select-Object Name, SamAccountName, Description, GroupScope |
    Export-Csv "groups-distribution-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Group Membership Export

```powershell
# Export specific group membership
Get-ADGroupMember -Identity "Domain Admins" -Recursive |
    Select-Object Name, SamAccountName, objectClass |
    Export-Csv "group-domain-admins-members-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export membership for multiple privileged groups
$PrivilegedGroups = @("Domain Admins", "Enterprise Admins", "Schema Admins", "Account Operators")
$AllMembers = @()

foreach ($Group in $PrivilegedGroups) {
    $Members = Get-ADGroupMember -Identity $Group -Recursive
    foreach ($Member in $Members) {
        $AllMembers += [PSCustomObject]@{
            Group = $Group
            MemberName = $Member.Name
            MemberSamAccountName = $Member.SamAccountName
            MemberType = $Member.objectClass
        }
    }
}

$AllMembers | Export-Csv "groups-privileged-all-members-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### User's Group Memberships

```powershell
# Export all groups a specific user is member of
$User = "jsmith"
Get-ADPrincipalGroupMembership -Identity $User |
    Select-Object Name, SamAccountName, GroupScope, GroupCategory |
    Export-Csv "user-$User-groups-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export group memberships for all users
$Users = Get-ADUser -Filter * -Properties MemberOf
$UserGroups = @()

foreach ($User in $Users) {
    $Groups = $User.MemberOf | ForEach-Object {
        (Get-ADGroup $_).Name
    }
    $UserGroups += [PSCustomObject]@{
        UserName = $User.Name
        SamAccountName = $User.SamAccountName
        Groups = $Groups -join "; "
    }
}

$UserGroups | Export-Csv "users-all-group-memberships-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Empty Groups

```powershell
# Find and export empty groups
$AllGroups = Get-ADGroup -Filter *
$EmptyGroups = @()

foreach ($Group in $AllGroups) {
    $Members = Get-ADGroupMember -Identity $Group
    if ($Members.Count -eq 0) {
        $EmptyGroups += $Group
    }
}

$EmptyGroups | Select-Object Name, SamAccountName, GroupScope, GroupCategory |
    Export-Csv "groups-empty-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

## File Naming Conventions

Use descriptive names with dates:
- `groups-all-2024-02-11.csv`
- `groups-security-2024-02-11.csv`
- `group-domain-admins-members-2024-02-11.csv`
- `groups-privileged-all-members-2024-02-11.csv`

## What to Ask Claude

Example questions for group analysis:
- "Identify privileged group memberships that may be excessive"
- "Are there any empty or unused groups?"
- "Which users have administrative access?"
- "Recommend a group structure for departmental access"
- "Are there any circular group memberships?"
- "Identify nested group memberships that could be simplified"
