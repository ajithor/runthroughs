a bunch of open ports
including ftp, smb, 80, etc
ftp has anon login - shows an info.txt, which reveals images$ is a hidden share un smb
That share images$ on smb, initially shows READ-ONLY access, but a put file.txt shows Write access is also available.
So we upload a xampp-compatible php rev shell (for windows) from https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/reverse/php_reverse_shell.php
listening on port, and opening /images/rshell.php gives us low-level shell
whoami /priv shows seImpersonate enabled

Tried running
`.\jpm.exe -l 1337 -c "{0868DC9B-D9A2-4f64-9362-133CEA201299}" -p rshell_w.exe -t *`
but it did not work
Had to go with Printspoofer64.exe
put Pintspooferexe - through smb first
Then in Windows machine,
`Printspoofer.exe -i -c cmd.exe`

---
To get the low-level usr password, the hint says the user is automatically logged in, which means the password is stored as plaintext in the registery,
`reg query “HKLM\SOFTWARE\microsoft\windows nt\currentversion\winlogon”`

To get the administrator password, It was present in C:\Installs in a .bat file
And the vnc.ini had the vnc password encrypted.
Had to use the http://aluigi.altervista.org/pwdrec/vncpwd.zip to decrypt the password