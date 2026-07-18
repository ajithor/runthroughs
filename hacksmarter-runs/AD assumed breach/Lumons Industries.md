We start with `hellyr : H3lenaR!2025`, and both DC and the ms01 "intranet" host available.
Begin with gathering users, asrep, kerbroasting, certif, bloodhound. Nothing
smbclient shows a writable share.
There, we also find a pdf and a link that leads us to intranet.lumos.hacksmarter web.
Gobustering this dint give much. Hellyr's creds did work here, but no functionalities.

So we graduate to llmnr poisoning. Initially, LLMNR poisoning doesnt work.

HINT TAKEN :  use a library-ms cve [[NTLM_theft and library-ms]]
```zsh
python exploit.py
	#enter ip
sudo responder -I tun0 -A
#drop the generated file into the share
```
In hindsight, this was not necessary, as we started to get hits on responder, just before we dropped the file into the share.

From this, we get hash for harmonyc, which we crack to get password.
Spraying did not reveal any entry points, however, the web portal did, with functionalities for synonymous xp_dirtree and to enable user.

We delete the files from the share, so we dont get more hits on responder, and do the dirtree part, and get a hash for IntranetSvc, crack and get password.

Nothing from sprays.
Bloodhound shows generic all over 6 users, 2 of whom, are laps admins. peterk (we later discover is disabled) and johns. We reset password for both, and cant do much with peterk. Looks same for johns, except johns has a winrm on intranet
None of our regular laps reading method works, pyLaps or Get-ADComputer

HINT TAKEN - use nxc ldap --module laps

With this, we get creds for `localadmin : ShoutUponGrapeSnareBroomCramp`

However, we cant cleanly authenticate as local admin through smb, psexec, or even secretsdump.
However, rdp worked. So, we add johns as a local admin, then use johns `nxc --sam --lsa`
From this, we get creds for hellye, who turns out, is a domain admin