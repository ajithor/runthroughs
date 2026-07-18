nmsp shows ports 22, 80 8089 open
port 80 seems default ish

Port 8089 is running flat press
Visiting the site, we see admin:password works for logging in. We also find, from "look for updates", the version of the flatpress is 1.2.1

With this, we lookup exploits for this, and we find an advisory, where it says "the uploader section has a remote file inclusion vuln, where one can upload malicious php file"

We upload a test.txt file as a control.
Rename `php-reverse-shell.php` to `rshell.txt.phtml`
`hexeditor rshell.txt.phtml` and match it with txt magic bytes, fix the `<?php tag`.

Now, while uploading, capture the req on burp and change the filetype to that of text

The file gets uploaded, and we can click on `media manager` to click on the file, and get a rev shell.
www-data!

---
```zsh
sudo -l
apt-get
#gtfo
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
#root!
```