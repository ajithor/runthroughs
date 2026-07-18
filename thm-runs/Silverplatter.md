initial scans showed ports 8080,80,22 open
port 80 - contact us - revealed username scriptkiddy at someplace called silverpeas
dirsearch on :8080/ showed existence of a page called silverpeas crm
when logging in, capture the req, and remove the password field from the req
This is vuln's exploit for silverpeas crm.

---
Once we get in, in my notifications, there is a mail, once u click on, it says ?id=5
seems like an idor. wen we made it ?id=6, we saw ssh login for tim
`ssh tim@IP`
`id` -- showed the user is a part of adm group
```text
`adm` group is a **system group** that grants its members **read access to system logs** stored in `/var/log`. It is primarily used for **monitoring and administrative purposes** without requiring full root privileges. Users in the `adm` group can view logs like `syslog`, `auth.log`, and `dmesg`
```
auth.log.2 had password for user tyler
su tyler
sudo -l showed all commands can be run with sudo
sudo -i -- root!

---
to get the initial foothold, alternatively
`cewl http://IP:80 > pass_list.txt`
`hydra -l scr1ptkiddy -P pass_list.txt 10.201.62.114 -s 8080 http-post-form "/silverpeas/AuthenticationServlet:Login=scr1ptkiddy&Password=^PASS^&DomainId=0:Login or password incorrect" -t 32`
