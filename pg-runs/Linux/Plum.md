nmap shows ports 22 and 80 open
port 80 runs pluXml CMS blog
searchsploit indicates existance of an rce for a lower version, but we havent yet determined the version.
Not many helpful things on gobuster.

Looking at the blogpage, we see an "Administration" link, that leads us to a login page, where admin:admin works

Tried editing a theme file to a php-rev-shell, but couldnt find its url
Then static page -> edit -> php-rev-shell -> save -> view static page = rev-shell!
www-data!

---
checking for suid files, we see exim4. exim4 is a mailing client.
Ideally, we have a few privescs for exim.
However, seeing from `ss -tulnp`, that port 25 is running locally, decided to check the mails in `/var/mail` and `/var/spool/mail`

That's where we find root's password
su root
root!