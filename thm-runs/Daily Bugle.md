Daily-Bugle initial scans show 22,80, 3306(mysql?) port
tcp : 80 spider.txt show an /administrator page, which appears to be running Joomla 3.7.0
Joomla 3.7.0 has a blind sqli vulnerability
sqlmap gave a dump of user jonah and a password hash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --fork=32
reveals the password to be spiderman123
can be either for ssh or the /administrator(doubtful)
it worked for the administrator

NOTE - here, had too look for joomlah vulerabilities. DINT
The walkthroughs showed how in /templates/protostar, there were editable pages like index.php, error.php etc.
Edited index.php to give reverse shell, and whoami-apache -- a low-level user
Looking for where joomlah stores passwords, it showed a configuratoin.php file.
had to look around to find it in /var/www/html/configuration.php
It had the password.
su jjameson (jjameson was the user folder from apache's /home, which was not readable)
once logged into jjameson, user.txt
Now, sudo -l showed yum as nopassword
gtfobins had a way to get a root shell from there
NOTE - had to copy-paste the contents line-by-line, as a user said it wasnt working if the whole code was pasted and ran
