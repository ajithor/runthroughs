Enumeration was indeed, the key
initial scans show 21(ftp), 80(joomla), 10000(webmin), 55007(ssh)
anonymous login was allowed for ftp
ls -l showed a log.txt, which was a dead end

did gobuster for /joomla, with medium.txt, but that did not contain /\_test
had to do with the wordlist, /usr/share/wordlists/dirb/common.txt
NOTE - always do gobuster with both these
the /\_test had a sar2html page, which, turns out was vulneerable to command injection at
```txt
http://<ipaddr>/index.php?plot=;<command-here> will execute 
the command you entered. After command injection press "select # host" then your command'
```
;ls there showed a log, which said ssh user basterd, and his password
`ssh basterd@IP -p 55007`
ls -la showed a backup.sh, which had user stoner, password
su stoner, password
SUID check showed find
gtfo bins, which find, cd /usr/bi
`./find . -exec /bin/sh -p \; -quit`
root!