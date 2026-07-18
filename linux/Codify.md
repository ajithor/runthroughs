initial scans show ports 21,80,300(nodejs)
redirects to codify.htb, put that to hosts file
website shows a vm2 library 3.9.16, which has a ready exploit available
replace command in the poc with mkfifo, and we have rev shell

---
in /var/www/contacts, there is a tickets.bd sqlite3 database file
move it to our machine,
`sqlite3 tickets.db`
`.dump`
shows us user joshua's bcrypt hah (possibly)
Send it to a file, get john to work (doesnt work)
$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
the $2a means bcrypt, so hashcat -m 3200
`hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt`
we get the password, spongebob1 for joshua

---
sudo -l shows we can run mysql_backup.sh script
When we take a look at this script, we see this takes bd_password input from user, and matches it with /root/.creds
But the way it does is vulnerable to pattern matching. 
`if [[ $DB_PASS == $USER_PASS ]]; then` it is using == instead of = inside of [[]], which compares absolute strings
Say, the actual password is `Passw0rd`, we can just input an asterics* and it will just work.
Condition1 -- $USER_PASS has to be n the right side of the ==
Condition2 -- $USER_PASS shouldnt be in ""
The next thing is, the way it passes the password to mysql is by providing /root/.creds and not the user-provided password.
This means, we can view the real password by using a process snooping tool such as pspy
Do note - pspy is 32-bit version. pspy64 is 64-bit version. get the one based on the uname -a of the target
Now, we need another ssh session. 1 to run the mysql_backup.sh, and the other for pspy64s
`./pspy64s`
`sudo /opt/scripts/mysql_backup.sh`
enter*
And on the pspy, we capture the password
Note - pspy was only able to view it because the /etc/ftab ability was set to not hideprocid=2
kljh12k3jhaskjh12kjh3
`su root` 
and root shell

---
tried with
`for i in $(seq 0 100); do ps -ef | gre mysql; sleep 0.5; done`
and ran the backup script in another sessh, but the password being sent is getting maked. so had to go with psy64s
Note - pspy64s is a linked binary, with pspy64, and used when size of the bin is an issue

