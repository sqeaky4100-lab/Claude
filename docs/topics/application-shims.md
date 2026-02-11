# Application Shims in Windows

## Overview

Application shims are a Windows Application Compatibility feature that allows the operating system to apply fixes to applications without modifying the application's code. Shims intercept API calls and modify the behavior to provide compatibility fixes for legacy applications.

## What Are Application Shims?

Shims are small pieces of code that sit between an application and the Windows API. When an application makes an API call, the shim can:
- Modify parameters sent to the API
- Change the return values from the API
- Emulate different OS versions
- Redirect file or registry operations
- Handle missing dependencies

## When to Use Shims

Use application shims when:
- Legacy applications won't run on newer Windows versions
- Applications require administrative privileges unnecessarily
- Applications have hard-coded paths that need redirection
- Applications expect specific OS versions
- Third-party applications can't be recompiled or patched
- Applications have known compatibility issues with Windows updates

## Application Compatibility Toolkit (ACT)

### Components

1. **Compatibility Administrator** - Create and manage shim databases
2. **Setup Analysis Tool** - Analyze application installers
3. **Standard User Analyzer** - Identify privilege elevation issues
4. **Internet Explorer Compatibility Test Tool** - Test web applications

## Common Shims

### 1. Version Lying Shims
Make the application think it's running on a different OS version.

- `Win7RTMVersionLie` - Reports Windows 7
- `Win8RTMVersionLie` - Reports Windows 8
- `Win10RTMVersionLie` - Reports Windows 10
- `VistaRTMVersionLie` - Reports Windows Vista

### 2. Privilege Shims
Handle applications that request unnecessary admin privileges.

- `RunAsInvoker` - Run with user's current privileges (no elevation)
- `RunAsAdmin` - Force application to run elevated
- `VirtualRegistry` - Redirect HKLM writes to HKCU
- `VirtualizeDeleteFile` - Redirect file deletions to virtual store

### 3. File System Shims
Handle file operations and paths.

- `RedirectEXE` - Redirect executable to different location
- `RedirectShortcut` - Redirect shortcuts
- `VirtualizeRegisterTypeLib` - Virtualize type library registration
- `CorrectFilePaths` - Fix hard-coded file paths

### 4. Compatibility Mode Shims
Apply multiple compatibility fixes in a package.

- `WinXPSP3` - Windows XP SP3 compatibility mode
- `Win7RTM` - Windows 7 compatibility mode
- `Win8RTM` - Windows 8 compatibility mode

## Creating a Custom Shim Database

### Step-by-Step Process

1. **Install Application Compatibility Toolkit**
   - Download from Microsoft
   - Install on administrative workstation
   - Launch "Compatibility Administrator (32-bit or 64-bit)"

2. **Create New Database**
   ```
   File > New > Database
   ```

3. **Add Application Fix**
   - Right-click "Applications" > New > Application Fix
   - Enter application name and vendor
   - Browse to application executable

4. **Select Compatibility Fixes**
   - Choose appropriate shims from the list
   - Common starting point: Version lie + privilege shim
   - Test one fix at a time for troubleshooting

5. **Test the Fix**
   - File > Install
   - Launch the application
   - Verify the fix works as expected

6. **Save and Deploy**
   - File > Save
   - Save as `.sdb` file
   - Deploy via Group Policy or manually

### Example: Making a Legacy App Work as Standard User

**Problem**: Application writes to Program Files directory, requires admin rights.

**Solution**: Apply VirtualizeRegisterTypeLib shim

```
1. Create new application fix in Compatibility Administrator
2. Target application: C:\Program Files\LegacyApp\app.exe
3. Select shim: VirtualizeRegisterTypeLib
4. Add matching file information to ensure it only applies to this app
5. Save as legacy-app-fix.sdb
6. Install: sdbinst.exe legacy-app-fix.sdb
```

## Deploying Shim Databases

### Method 1: Manual Installation

```batch
REM Install shim database
sdbinst.exe -q "C:\ShimDatabases\myfix.sdb"

REM Uninstall shim database
sdbinst.exe -u "C:\ShimDatabases\myfix.sdb"

REM List installed databases
sdbinst.exe -l
```

### Method 2: Group Policy

1. Create a share for shim database files
2. Configure Group Policy:
   ```
   Computer Configuration > Policies > Administrative Templates > Windows Components > Application Compatibility
   ```
3. Enable: "Turn off Application Compatibility Engine"
   - Set database path: `\\server\share\myfix.sdb`

### Method 3: SCCM/Intune Deployment

- Package the `.sdb` file
- Create installation script using `sdbinst.exe`
- Deploy to target devices
- Verify installation through registry or sdbinst -l

## PowerShell Automation

### Install Shim Database via PowerShell

```powershell
# Install shim database
$sdbPath = "C:\ShimDatabases\appfix.sdb"
Start-Process -FilePath "sdbinst.exe" -ArgumentList "-q `"$sdbPath`"" -Wait -NoNewWindow

# Verify installation
$installed = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\InstalledSDB\*" |
    Where-Object { $_.'DatabasePath' -like "*appfix.sdb" }

if ($installed) {
    Write-Host "Shim database installed successfully" -ForegroundColor Green
} else {
    Write-Host "Installation failed" -ForegroundColor Red
}
```

### Check Active Shims

```powershell
# List installed shim databases
$shimsKey = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\InstalledSDB"
Get-ChildItem $shimsKey | ForEach-Object {
    $db = Get-ItemProperty $_.PSPath
    [PSCustomObject]@{
        DatabaseID = $_.PSChildName
        DatabasePath = $db.DatabasePath
        DatabaseDescription = $db.DatabaseDescription
    }
}
```

## Troubleshooting Shims

### Enable Application Compatibility Logging

```powershell
# Enable compatibility logging
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags" -Name "LogFlags" -Value 4
```

Log location: `%SystemRoot%\AppPatch\Shim\Logs`

### Check Which Shims Are Applied

Use Process Monitor (Sysinternals):
1. Launch Process Monitor
2. Filter for your application
3. Look for "AppHelp" and "Shim" operations

### Common Issues

**Issue**: Shim not applying to application
- Check matching files (name, version, size)
- Verify database is installed
- Ensure executable path matches
- Check for 32-bit vs 64-bit mismatch

**Issue**: Application still doesn't work
- Try different shim combinations
- Check application event logs
- Use Standard User Analyzer to identify issues
- Consider combining multiple shims

**Issue**: Shim causes new problems
- Test shims individually
- Use minimal set of shims needed
- Check for conflicts with other compatibility settings

## Best Practices

1. **Test Thoroughly**
   - Test in non-production environment first
   - Test with standard user accounts
   - Test all application features

2. **Use Specific Matching**
   - Match by file version and checksum
   - Avoid matching by name only
   - Prevents shim from applying to wrong applications

3. **Document Shims**
   - Document why each shim was applied
   - Note the problem it solves
   - Track which systems have the shim

4. **Minimize Shim Use**
   - Shims should be temporary solutions
   - Work with vendors for proper fixes
   - Update applications when possible

5. **Security Considerations**
   - Be cautious with RunAsAdmin shim
   - Test security implications
   - Ensure shims don't bypass security features

## Alternative Solutions

Before using shims, consider:

1. **Application Updates**: Check if vendor has newer compatible version
2. **Virtualization**: Run in VM or container with older OS
3. **File/Registry Permissions**: Grant specific permissions instead of elevation
4. **Application Packaging**: Repackage with proper settings
5. **Remote Desktop Services**: Host application on server

## References

- Microsoft Application Compatibility Toolkit Documentation
- Windows Assessment and Deployment Kit (ADK)
- Sysinternals Process Monitor
- Microsoft Compatibility Administrator Guide
