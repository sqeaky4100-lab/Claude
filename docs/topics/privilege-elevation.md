# Privilege Elevation in Windows Environments

## Overview

Privilege elevation is a critical aspect of Windows security management. This document covers User Account Control (UAC), admin rights management, and best practices for controlled elevation.

## User Account Control (UAC)

### What is UAC?

User Account Control is a security feature that helps prevent unauthorized changes to the operating system. When a program tries to make changes that require administrator-level permissions, UAC prompts the user for approval.

### UAC Levels

1. **Always notify** - Highest security, prompts for all elevation attempts
2. **Notify only when programs try to make changes** - Default setting
3. **Notify only when programs try to make changes (no desktop dimming)** - Less secure
4. **Never notify** - Least secure, not recommended

### Configuring UAC via Group Policy

Key GPO settings location: `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options`

Important UAC policies:
- `User Account Control: Admin Approval Mode for the Built-in Administrator account`
- `User Account Control: Behavior of the elevation prompt for administrators`
- `User Account Control: Behavior of the elevation prompt for standard users`
- `User Account Control: Run all administrators in Admin Approval Mode`

## Elevation Strategies

### 1. Run As Administrator

**Manual Elevation:**
- Right-click executable → "Run as administrator"
- Creates elevated process with full admin token

**Programmatic Elevation:**
```powershell
# Launch elevated process from PowerShell
Start-Process powershell -Verb RunAs
```

### 2. Scheduled Tasks

Running tasks with elevated privileges without UAC prompts:
- Create scheduled task with highest privileges
- Task runs in background with elevated token
- No UAC prompt for scheduled tasks

**Example PowerShell:**
```powershell
$Action = New-ScheduledTaskAction -Execute "C:\Scripts\maintenance.ps1"
$Trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName "Maintenance" -Action $Action -Trigger $Trigger -RunLevel Highest
```

### 3. Windows Services

Services run with SYSTEM or designated service accounts:
- No UAC interaction required
- Higher privilege than user accounts
- Good for background automation

### 4. RunAs with Credential Management

```powershell
# Store credentials securely
$Credential = Get-Credential
$Credential.Password | ConvertFrom-SecureString | Out-File "C:\secure\cred.txt"

# Use stored credentials
$Password = Get-Content "C:\secure\cred.txt" | ConvertTo-SecureString
$Credential = New-Object System.Management.Automation.PSCredential("DOMAIN\AdminUser", $Password)
Start-Process -FilePath "program.exe" -Credential $Credential
```

## Best Practices

### Least Privilege Principle

1. **Standard User Accounts**: Users should operate with standard accounts for daily tasks
2. **Separate Admin Accounts**: Admins should have separate privileged accounts for administrative tasks
3. **Just-In-Time (JIT) Administration**: Grant elevated access only when needed, revoke afterward

### Application Whitelisting

Use AppLocker or Windows Defender Application Control to:
- Restrict which applications can run with elevation
- Prevent unauthorized elevation attempts
- Control execution based on publisher, path, or hash

### Monitoring and Auditing

Enable auditing for privilege use:
```
Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies > Privilege Use
```

Monitor these Event IDs:
- **4624**: Successful logon (check for elevation)
- **4672**: Special privileges assigned to new logon
- **4688**: Process creation (with command line auditing)

## Common Issues and Solutions

### Issue: Applications Don't Work with Standard User Rights

**Solutions:**
1. **Application Shims**: Use compatibility fixes (see Application Shims documentation)
2. **File/Registry Permissions**: Grant specific permissions to needed resources
3. **Virtualization**: Enable UAC virtualization for legacy apps

### Issue: Too Many UAC Prompts

**Solutions:**
1. **Review software**: Update or replace software requiring frequent elevation
2. **Application Manifest**: Developers should properly configure manifests
3. **Scheduled Tasks**: Move routine elevated tasks to scheduled tasks
4. **Don't disable UAC**: Weakens security significantly

### Issue: Silent Elevation Needed for Specific Applications

**Solutions:**
1. **Application Elevation Policy**: Configure via Group Policy
   - `Computer Configuration > Policies > Windows Settings > Security Settings > Application Control Policies > AppLocker`
2. **Trusted Applications**: Use AppLocker to allow specific apps to elevate silently
3. **Custom Solutions**: Use wrapper scripts or scheduled tasks

## Security Considerations

### Risks of Disabling UAC

- Malware can gain system-level access more easily
- No protection against unauthorized system changes
- Compliance issues (many frameworks require UAC)
- Silent installation of unwanted software

### Protected Administrators Group

Consider using Protected Users security group:
- Forces Kerberos authentication
- Prevents credential caching
- Restricts certain dangerous operations

## PowerShell Elevation Check

```powershell
# Check if current PowerShell session is elevated
$isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

if ($isAdmin) {
    Write-Host "Running with administrator privileges" -ForegroundColor Green
} else {
    Write-Host "Running as standard user" -ForegroundColor Yellow
    Write-Host "Restart PowerShell as administrator for elevated tasks"
}
```

## References

- Microsoft Docs: User Account Control
- Microsoft Security Baselines
- CIS Windows Benchmarks
- NIST Guidelines for Windows Security
