nmap shows ports 21 ftp anon, 80, 135rpc, 139 smb, 445 smb, 9998 web, 17001 remoting
deep scan reveals port 5040, 
We start with ftp `ftp IP`
We dont see anything useful, except an admin user, and a clamAV database

nothing on webpage 80
port 9998 shows smartermail login page
no default creds login

we lookup public exploits, find one
https://www.exploit-db.com/exploits/49216

change IP, and we're NT authority
