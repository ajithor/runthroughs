ports 22 and 80 open
gobuster on port 80 shows /election and /phpmyadmin.
Couldnt get it with default creds on phpmyadmin

/election dint seem like it held any promise
So, we gobuster /election -x php
We find /card.php /admin /logs.php
/card.php has a binary string sequence, which upon converting to ascii twice, gives us creds
user:1234 pass:Zxc123!@#

Using these creds to login to /admin, we see it is creds for the user "love"
In the webpage, settings->system info -> logging -> view logs leads us to /admin/logs.php

 `[2020-01-01 00:00:00] Assigned Password for the user love: P@$$w0rd@123` ssh

we see we have adm group access, so we can see stuff from /var/log
Looking for SUID binaries, we find `/usr/local/Serv-U/Serv-U`

searchsploit serv-U
searchsploit -m multiple/local/47173.sh

transfer it over to target, and run it, we get root!