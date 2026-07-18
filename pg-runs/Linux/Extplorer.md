nmap shows ports 22 and 80 open
80 seems to be a work in progress wps site
but `wpscan --url $IP` says the site isnt running wordpress
gobuster initially shows only the wp endpoints, then errors out
So we look into the wp stuff, to no avail

HINT TAKEN : filemanager endpoint
upon rerunning gobuster, we find the endpoint, and are able to login with admin:admin

in the file manager, we're able to create/delete a php file
So that is file upload, where we upload php-reverse-shell.php

www-data!

---
HINT TAKEN in filemanager/config/ there is a hidden file, .htusers.php where dora's hash is stored. crack it for dora's password. should've seen this in the webpage itself!

we crack it to be doraemon
 su dora --> doraemon
dora!

---
dora is a member of disk group. which means she can mount any filesystem. if the filesystem just to happens to be rooted on "/", we can access everything. even /etc/shadow

```
df -h
#note the filesystem mounted on /
debugfs /dev/mapper/ubuntu--vg-ubuntu--lv

#now test if it is readonly, or writable as well
mkdir test 
#seems read-only

cat /etc/shadow
```
we get root's password hash
`root:$6$AIWcIr8PEVxEWgv1$3mFpTQAc9Kzp4BGUQ2sPYYFE/dygqhDiv2Yw.XcU.Q8n1YO05.a/4.D/x4ojQAkPnv/v7Qrw7Ici7.hs0sZiC.`

john it, crack it as "explorer"

su root
root!

---
---
---
since we had both /etc/passwd and /etc/shadow, we can use john unshadow to crack 
But it is ycrypt, so we couldnt really crack it
moving on...

