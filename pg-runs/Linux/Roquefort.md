nmap shows ports 21 ssh, 22 ssh, 3000 web, 2222 ssh open

port 21-
locked down, no anon access

port 3000 - 
Gitea 1.7.5
for login, default creds dont work, but there is a signup page

searchsploit shows an exploit that should give rce, if we have valid creds
So we just create an account
poop:pooper

run the exploit, and we have shell
chloe!

---
we see 3306 mysql is lisening on internal port
So we run ps -ef to see wagwan
we find `/usr/local/bin/gitea web --config /etc/gitea/app.ini` 
upon viewing the file, we find creds to mysql
```txt
DB_TYPE  = mysql
HOST     = 127.0.0.1:3306
NAME     = giteadb
USER     = gitea
PASSWD   = 7d98afcbd8a6c5b8c2dfb07bcbe29d34
SSL_MODE = disable
PATH     = data/gitea.db
```
--an md5 hash
but it was uncrackable using cracksation, and rockyou

HINT TAKEN : /usr/local/bin writable folder existing in $PATH revealed by linpeas. So, we just need to find some binary in crontab, whose real-path isnt used, and which comes after the /usr/local/bin in PATH variable order
We then write a malicious binary with that name "run-parts" in /usr/local/bin
and get a rev shell!

Twist - appearently, the run-parts command in `cd / && run-parts --report /etc/cron.hourly` line, which is essentially a standard part of crontab, is located at /bin/run-parts

So, we can just write a binary in our writable path with that name
