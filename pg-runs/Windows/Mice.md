deep nmap shows 1978, 1979, 1980 (remote mouse) winrm, 7680

Couldnt exactly deduce the version for remote mouse, but found an exploit, that works
`windows/remote/46697.py`
Verify it works in the payload line with a wget, and listen on port 80, it works

send over nc once, run it next
divine!

---
```powershell
#in \users\divine
findstr /SIM /C:"pass" *.ini *.cfg *.config *.xml
#we get 2 files
AppData\Roaming\FileZilla\filezilla.xml
AppData\Roaming\FileZilla\recentservers.xml
#The second one has base64-encoded pass which comes to ControlFreak11
```
Since we have rdp, we try it for administrator, but doesnt work. It shouldve been for divine anyways
We had seen a remot mouse GUI PE earlier on windows/local/50047.txt
following that,
we get admin! 