Initial scans show port 22,80,445
gobuster on port 80 - robots.txt, license.txt
roots had path to first flag
license had password base64 for user elliot
viewing some site showed wps
so went to wp-login - tried elliot-pass, worked.
Appearances -> editor -> 404.php --> copy the reverse-shell-php.php code into it
visit ip/404.php, and we have reverse shell!
user is daemon, and we find the second key unaccessible in the home dir.
Another file has other user's md5 hash
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt --force
gave pass abcd....xyz
su, and got the 2nd flag
SUID perms were set for nmap
gtfobins
nmap --interactive
sh!
root shell!