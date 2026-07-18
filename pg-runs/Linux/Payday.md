nmap shows 22,80,110 pop3,139 smb,143 imap,445 smb,993 ssl pop3,995 ssl imap

we find the webpage, and the search bar makes it look like an lfi ?target. it wasnt
HINT TAKEN : google for cs cart lfi exploit, and we find it
`GET /classes/phpmailer/class.cs_phpmailer.php?classes_dir=/etc/passwd%00 HTTP/1.`
gives us etc/passwd
Now, we try to take patrick's id_rsa. but it isnt there, neither is id_dss

So we proceed to seach exploits for ca cart v1.3.3
it says use a php rev shell, call it shell.phtml, upload via file manager, visit target/skins/shell.phtml
In our case, it is template editor
www-data!

---
Once we get shell, there's no privesc appearant
HINT TAKEN : su patrick --> password patrick

then sudo -l shows we can do everything, sudo -i
root!

---
---
---
alternatively, we couldve used the unauthenticated FLI to read the passwd file, then brute-forced ssh as patrick
`nxc ssh 192.168.189.39 -u patrick -p /usr/share/wordlists/rockyou.txt`