Initial scans show ports 22,80 open
80 has a simple apache server running, nothing interesting
gobuster shows /content, which seems to say SweetRice CMS
Priliminary reseach showed existence of backup disclosure vuln at 
```txt
http://localhost/inc/mysql_backup
```
Found a mysql bacup file, which contained the user - Manager, and hashed password, md5
md5 decrypter helped arrive at the password, Password123
Further gobuster no /content showed existance of /content/as, a login page

Logged into the CMS with this info
Another CVE showed how SweetRice 1.5.1 allows file upload at media-center
/as/?type=media_center&mode=upload
reverse shell! www-data

The /home/it-guy/ had a perl backup script, which pointed to another file with bad permissions, and we edited to give us root remote shell!