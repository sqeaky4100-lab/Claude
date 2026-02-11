# Windows Environment Security Audit Checklist

Use this checklist as a guide for conducting security audits of Windows environments.

## User Account Security

### Password Policies
- [ ] Minimum password length meets requirements (12+ characters recommended)
- [ ] Password complexity enabled
- [ ] Password history prevents reuse (24+ passwords recommended)
- [ ] Maximum password age set (90 days or less recommended)
- [ ] Account lockout policy configured
- [ ] Account lockout duration appropriate

### User Account Review
- [ ] No accounts with passwords that never expire (except service accounts)
- [ ] No disabled accounts with administrative privileges
- [ ] No inactive accounts (90+ days)
- [ ] No shared accounts
- [ ] No unnecessary privileged accounts
- [ ] Service accounts use managed service accounts where possible
- [ ] No users with blank passwords

## Group Membership Review

### Privileged Groups
- [ ] Domain Admins membership justified and documented
- [ ] Enterprise Admins membership justified and documented
- [ ] Schema Admins membership justified and documented
- [ ] Backup Operators membership reviewed
- [ ] Account Operators membership reviewed
- [ ] No nested administrative groups unless necessary

### Access Groups
- [ ] Group membership follows least privilege principle
- [ ] Empty groups identified and removed
- [ ] Group naming conventions followed
- [ ] Group descriptions are current and accurate

## Group Policy Security

### Password and Account Policies
- [ ] Default Domain Policy enforces strong passwords
- [ ] Account lockout policy enabled
- [ ] Password policies applied consistently

### Security Settings
- [ ] User Account Control (UAC) enabled
- [ ] Windows Firewall enabled
- [ ] Windows Defender enabled (if applicable)
- [ ] Audit policies configured
- [ ] Software restriction policies or AppLocker configured
- [ ] Remote Desktop restricted appropriately

### Administrative Templates
- [ ] Auto-play disabled for removable media
- [ ] Guest account disabled
- [ ] Anonymous SID enumeration disabled
- [ ] LAN Manager authentication level set appropriately
- [ ] Network security settings hardened

## Privilege Elevation

### UAC Configuration
- [ ] UAC enabled for all users
- [ ] Admin Approval Mode enabled
- [ ] Elevation prompts secure desktop enabled
- [ ] Standard users cannot elevate

### Elevation Monitoring
- [ ] Administrative logons audited
- [ ] Special privileges auditing enabled
- [ ] Unnecessary elevation identified

## Audit and Logging

### Audit Policies
- [ ] Account logon events audited
- [ ] Account management audited
- [ ] Logon events audited
- [ ] Object access audited (as needed)
- [ ] Policy change audited
- [ ] Privilege use audited
- [ ] System events audited

### Log Management
- [ ] Security log size adequate
- [ ] Logs retained for required period
- [ ] Log collection configured (SIEM)
- [ ] Critical security events monitored

## Application Security

### Application Deployment
- [ ] Software deployment via GPO documented
- [ ] Only approved software can be installed
- [ ] Application compatibility shims reviewed
- [ ] Unnecessary software removed

### Updates and Patches
- [ ] Windows Update configured via GPO
- [ ] Patch compliance monitored
- [ ] Critical updates deployed timely

## Network Security

### Network Access
- [ ] Network shares documented and reviewed
- [ ] Anonymous access disabled
- [ ] Remote access restricted
- [ ] RDP security hardened
- [ ] Null session enumeration disabled

## Compliance

### Documentation
- [ ] Security policies documented
- [ ] Change management process followed
- [ ] Incident response plan exists
- [ ] Backup and recovery tested

### Standards Compliance
- [ ] CIS Benchmarks reviewed
- [ ] Industry-specific compliance (HIPAA, PCI-DSS, etc.)
- [ ] Security baseline applied
- [ ] Regular security assessments scheduled

## Notes

Document findings, risks, and recommendations here:

```
[Your notes]
```

## Recommendations

List prioritized recommendations:

1. 
2. 
3. 

## Next Steps

- [ ] Address critical findings
- [ ] Schedule follow-up audit
- [ ] Update documentation
- [ ] Report to management
