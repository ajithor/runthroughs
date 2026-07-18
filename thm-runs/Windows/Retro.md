Initial scans show port 80 and 3389(RDP)
gobuster on 80 showed an endpoint, /retro, which turns out to be a WPS blog site. In one of the comments, the user Wade wrote the password.
This could be for WPS login, or the RDP
We try for both, and it works for both.
Apparently, the WPS wont give you a shell, tho

---
We proceed with rdp, and the user.txt is right there
No SeImpersonate or anything
But there was an exe in the recycle bin
There was also a CVE that the user was looking for, in chrome history
The CVE says if an application is able to Trigger Windows Certificate Dialog, we can open the CA link, on a web browser (like IE) and save (ctrl+s) then go to C:\Windows\Program32\cmd.exe and it would give you elevated priv cmd.

Note - in Windows2016, intentionally introduced bug is when you wanna open the link, it deosn't show any browser options. To solve that, open both chrome and IE, and ONLY then run the exploit.

So we follow the steps. Restore the exe, run, click on more info - CA - select browzer.
ctrl+s -> c:\Windows\Programs32\ click cmd.exe, open --> root shell!