smb, rpc, ldap locked down
There's a "secure web browser" on port 8080
when we put our IP, we get a request on python server
tried to send a shell, php shell, etc. it just displays the text
HINT TAKEN :  do the same, with responder running!

we get NTLM challenge-response for enox, which cracks as california
no extra readable shares, no extra users, except for admin
winrm is pwned
enox!

---
from there, due to no obvious paths in winEPAS, we run bloodhound, and it shows we have 2 paths
Frist is we have write on a Default Domain Policy, but dacledit doent work
Second is, as a member of "Web Admins", we can read the gmsa password of svc_apache$

`nxc ldap 192.168.110.165 -u enox -p california --gmsa`
gives us NTLM hash of svc_apache$ and we're able to winrm with it
svc_apache$

---
From here, svc_apache$ has GenericAll on DC01, which means an RBCD attack
But this failed at impacket-rbcd stetp, saying insufficient rights. Damn you, bloodhound!

HINT TAKEN : swisskey repo has a way to privesc when there is only SeRestore present
1. Launch PowerShell/ISE with the SeRestore privilege present.  
2. Enable the privilege with [Enable-SeRestorePrivilege](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1)).  
3. Rename utilman.exe to utilman.old  
4. Rename cmd.exe to utilman.exe  
5. Lock the console and press Win+U

Following this,
```
cd \windows\system32
ren Utilman.exe Utilman.old
ren cmd.exe Utilman.exe
```

on kali, `rdesktop IP`
windows+u
cmd.exe as NT Authority!!

---
---
---
Alternatively, we can use https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe to run commands as NT Authority, whenever we have seRestore available
