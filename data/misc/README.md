# Miscellaneous Files Directory

Store other Windows management related files that don't fit into specific categories.

## File Types

This directory can contain:
- Event log exports
- Registry exports
- Security audit reports
- Compliance reports
- Network configuration files
- System information exports
- Custom reports
- Documentation
- Screenshots
- Other diagnostic files

## Common File Formats

- `.evtx` - Event log files
- `.reg` - Registry exports
- `.txt` - Text files
- `.pdf` - PDF reports
- `.docx` - Word documents
- `.xlsx` - Excel spreadsheets
- `.png`, `.jpg` - Screenshots

## Useful Export Commands

### Event Logs

```powershell
# Export Security event log
Get-EventLog -LogName Security -Newest 1000 |
    Export-Csv "security-events-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export Application event log
Get-EventLog -LogName Application -Newest 1000 |
    Export-Csv "application-events-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Export System event log
Get-EventLog -LogName System -Newest 1000 |
    Export-Csv "system-events-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### System Information

```powershell
# Get computer information
Get-ComputerInfo | Export-Clixml "computerinfo-$(Get-Date -Format 'yyyy-MM-dd').xml"

# Get installed software
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Export-Csv "installed-software-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

### Network Configuration

```powershell
# Get network adapter configuration
Get-NetAdapter | Export-Csv "network-adapters-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation

# Get IP configuration
Get-NetIPAddress | Export-Csv "ip-configuration-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

## What to Ask Claude

- "Analyze these event logs for security issues"
- "Review this system configuration"
- "What network settings might be causing issues?"
- "Interpret these diagnostic results"
