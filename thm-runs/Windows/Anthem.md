Portscans showed ports 80 and 3389(RDP) to be open
robots.txt had a password in it
There was a poem regarding the admin, which lead to Solomon Grundy
Another user who was John Doe had email JD@anthem.com
So guessed SG@anthem.com with the password from robots.txt, logged in to the website

---
Used the same password, with username SG to log into remote desktop
`xfreerdp3 /u:SG /p:UmbracoIsTheBest! /v:10.201.124.236:3389 +clipboard`

found the users.txt right away
There was a hidden dir in C:\ , the backup
And there was a file, with admin password, non-readable to SG

Through walkthroughs, we found we can add access to ourselves to read the file. Wah!
`icacls restore.txt /grant Everyone:F`

powershell - open as admin
root!
