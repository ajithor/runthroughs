nmap shows ports 22, 23 web, 80 web?, 3306 mysql

webpage on 80 says "under construction"
`feroxbuster -u http://192.168.244.41 -t 64 -v -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`
shows a /test page, where we can search for photos
seems like Dir traversal/LFI
also, pulls a test/robots, which shows /test/uploaded/ among others
maybe a file upload vuln?

while trying something on burp, we find zenphoto version 1.4.1.4
looking on searchsploit, we find php/webapps/18083.php, a remote file creation vulnerability

Dint understand much of the working, there's a vulnerable endpoint, where we inject a command

`php ajax_create_folder.php 192.168.244.41 /test/ `
gave a shell
we quickly find it is not doing any cd
we send over the mkfifo 1 liner to /dev/shm, and make it give us a shell
wayy better

---
www-data
none of the privesc work, nothing!, nothing on linpeas as well
HINT TAKEN : turns out, it was a stupid kernel exploit

https://www.exploit-db.com/exploits/15295
if compiling on kali, 
```zsh
sudo apt-get install gcc-multilib -y
gcc 15285.c -o 15285 -m32
```
compiling on target just easier
root!