nmap shows ports 21, 22, 80
ftp empty
gobuster on port 80 shows endpoints /news /monitoring /webmail

Looking at /news, we find the possibility of a user, Otis

/monitoring leads to a login page, where otis:123456 creds work
This page sends an automated ping to any IP we mention to check if the host is up are not

/webmail has another login, where otis:123456 works
No mails in the inbox so far.

But when we enter an invalid IP in the /monitoring and wait for the ICMP to do its job, once it detects the IP is invalid, we get a mail

The understanding so far, is that when the invalid IP is detected, /monitoring sends a mail to /webmail by extracting data from its database
To test the sqli on the /monitoring's database, we can use `/usr/share/wfuzz/wordlist/vulns/sql_inj.txt` . So capture a request on burp, send to intruder, and send requests. Then wait for it, and check mails

From there, we find `" or 1=1 #` is the magical break
And so we get the database names, tables, users and passwords
Taking elliot's encrypted password, we get new ssh creds
elliot:elliot123
```zsh
ssh elliot@IP

ls -la
#shows .mozilla directory
```
Whenever we see the mozilla folder, if there are cached creds, it can be recovered using these 4 files `logins.json cert9.db cookies.sqlite key4.db`
```zsh
ls .mozilla/firefox/esmhp32w.default-default | grep -E "logins.json|cert9.db|cookies.sqlite|key4.db"
```

transfer these to kali, clone a repo `https://github.com/unode/firefox_decrypt`
```zsh
git clone https://github.com/unode/firefox_decrypt
/opt/firefox_decrypt/firefox_decrypt.py .
#select option 2
gives us the root password
su root
root!
```
