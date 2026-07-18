nmap shows ports 22, 80 open
Visiting the webpage, we're greeted with a codoforum login, where admin:admin works

Nothing too interesting on the admin's profile, except an opportunity to edit the profile pic.

Looking at searchsploit, we see there's a file upload -> rce vulnerability, in a different location.
So running gobuster, we find an /admin endpoint, where again, admin:admin works.
According to the exploit, the `/admin/index.php?page=config` has a place to upload an image.
There, first, I changed the "allowed extensions" to include a php extension.
Then uploaded php-reverse-shell.php.
Following the exploit, we visit `IP/sites/default/assets/img/attachments/rev.php`
Gives a reverse shell.
www-data!

---
digging around the www-data home file system, in `/var/www/html/sites/default/config.php`, we find a password `FatPanda123`
`su root #FatPanda123` gives us
root!ex