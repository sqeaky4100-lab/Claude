# GPO Reports Directory

Store Group Policy Object (GPO) reports in this directory for analysis.

## Supported File Types

- `.html` - HTML reports from Get-GPOReport or GPMC
- `.xml` - XML reports from Get-GPOReport
- `.csv` - CSV exports of GPO data

## How to Generate GPO Reports

### Using PowerShell

```powershell
# Generate HTML report of all GPOs
Get-GPOReport -All -ReportType Html -Path "gpo-report-all.html"

# Generate XML report for specific GPO
Get-GPOReport -Name "Default Domain Policy" -ReportType Xml -Path "default-domain-policy.xml"

# Generate report for GPO linked to specific OU
$GPOs = Get-GPInheritance -Target "OU=Workstations,DC=contoso,DC=com"
foreach ($GPO in $GPOs.GpoLinks) {
    Get-GPOReport -Guid $GPO.GpoId -ReportType Html -Path "$($GPO.DisplayName).html"
}
```

### Using gpresult Command

```batch
REM Generate HTML report for current computer and user
gpresult /h gpresult-report.html

REM Generate verbose text report
gpresult /v > gpresult-verbose.txt

REM Generate report with specific scope
gpresult /Scope Computer /h gpresult-computer.html
gpresult /Scope User /h gpresult-user.html
```

### Using Group Policy Management Console (GPMC)

1. Open Group Policy Management Console
2. Navigate to the GPO you want to report on
3. Right-click the GPO → Settings → Save Report
4. Choose HTML or XML format
5. Save to this directory

## File Naming Conventions

Use descriptive names with dates:
- `default-domain-policy-2024-02-11.html`
- `workstation-security-gpo-2024-02-11.xml`
- `server-baseline-gpo-2024-02-11.html`
- `all-gpos-report-2024-02-11.html`

## What to Ask Claude

Example questions for GPO analysis:
- "Review the GPO report and identify any security concerns"
- "What password policies are configured in this GPO?"
- "Are there any conflicting settings between these GPO reports?"
- "Recommend improvements to this security policy"
- "Which software restrictions are in place?"
